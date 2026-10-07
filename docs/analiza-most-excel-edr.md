# Most do Excela a oprogramowanie EDR — analiza

Stan kodu: commit `8ff1a5f` (2026-10-06 14:59). Numery linii dotyczą `PivotStudio.py` w tej wersji.

## W skrócie

- **Problem.** „Sesja Excela” uruchamia Excela w sposób, który korporacyjny EDR rozpoznaje jako podręcznikowy atak typu
  „fileless PowerShell”. Do tego wzorca należą: ukryty `powershell.exe` z `-EncodedCommand`, skrypt przekazany przez stdin
  i uruchomiony przez `[ScriptBlock]::Create`, kompilacja C# w locie (`Add-Type` z `DllImport`) oraz start Excela przez COM.
  W zgłoszonym incydencie EDR zabił proces Pivota i jego PowerShella. Plik `PivotStudio.py` trafił do kwarantanny.
- **Zasięg.** To jedyne miejsce w aplikacji, które uruchamia PowerShell. Podgląd, import i edycja XLSX/XLSM czytają plik
  jako ZIP/XML w Pythonie i nie uruchamiają ani PowerShella, ani Excela.
- **Kiedy powstało.** Most do Excela pojawił się w repozytorium 2026-10-06 o 13:06 (`a1e322a`). Bootstrap przez stdin
  i `ScriptBlock::Create` doszedł o 13:50 (`11c1c90`), a filtr komunikatów OLE (`CoRegisterMessageFilter`) o 14:59 (`8ff1a5f`).
- **Kierunek naprawy.** Excelem ma sterować osobny proces Pythona przez COM (comtypes), uruchamiany tak samo jak istniejący
  `--worker`. Do tego wyłącznik mostu w ustawieniach. **Żaden projekt nie gwarantuje, że EDR przestanie reagować.**
  Interpreter skryptów sterujący Office'em przez DCOM nadal może być oceniany jako podejrzany, więc do użycia na komputerach
  firmowych potrzebna jest decyzja i ewentualny wyjątek od działu IT.
- **Osobny błąd (bezpieczeństwo).** W oknie połączenia Oracle klawisz Tab przenosi z pola „Użytkownik” do niemaskowanego
  pola „Schemat (opcjonalny)”, a nie do pola „Hasło”. Wpisanie „użytkownik, Tab, hasło, Enter” zapisuje hasło jawnym tekstem
  w projekcie i w kilku innych miejscach. Szczegóły w ostatniej części.

## 1. Co dzieje się po kliknięciu „Uruchom sesję Excela”

Sesję uruchamia wyłącznie przycisk w oknie „Excel: makra i PDF” (`ExcelSessionController` tworzony tylko w `:10642`).
Menu, pasek „Otwórz w Excelu…” i przyciski skopiowane ze skoroszytu otwierają to okno, ale same niczego nie uruchamiają.

1. `excel_availability()` (`:4327`) czyta z rejestru CLSID `Excel.Application` i sprawdza, czy istnieje Windows PowerShell 5.1 (`:4331`).
2. Prywatna kopia skoroszytu trafia do `%LOCALAPPDATA%\PivotStudio\excel-sessions\session-*\` (`:4828-4856`). Kopiuje ją
   `CopyFileExW`, który zachowuje Mark-of-the-Web. Obok powstaje `session.json`.
3. Python sprawdza listę procesów `EXCEL.EXE` (Toolhelp32, `:4342-4362`).
4. `subprocess.Popen` uruchamia `powershell.exe -NoLogo -NoProfile -NonInteractive -Sta -EncodedCommand <base64>`
   z flagą `CREATE_NO_WINDOW` (`:4858-4862`). Zakodowany tekst to stały stager `EXCEL_SESSION_BOOTSTRAP` (`:4412`):
   czyta jedną linię base64 ze stdin i uruchamia ją przez `[ScriptBlock]::Create`.
5. Do stdin trafia base64 stałego skryptu `EXCEL_SESSION_POWERSHELL` (`:4471-4767`, `:4864`). Skrypt nigdy nie jest zapisywany na dysku.
6. Skrypt wykonuje `Add-Type` z C# (`:4652-4670`). Są w nim `DllImport` z `user32!GetWindowThreadProcessId`
   i `ole32!CoRegisterMessageFilter` oraz własny `IOleMessageFilter`. W PowerShell 5.1 `Add-Type` kompiluje kod przez
   `csc.exe` (proces potomny) i ładuje tymczasowy DLL z `%TEMP%`.
7. `New-Object -ComObject Excel.Application` (`:4674`). `EXCEL.EXE` uruchamia usługa DCOM (svchost), a nie PowerShell.
8. Następuje uzgodnienie PID/HWND z Pythonem (`created` → `ExcelOwnedProcess` → `attach`, `:4679-4680`, `:4892-4898`).
   Dopiero potem są `AutomationSecurity = 2` i `Workbooks.Open` (`:4682-4686`). `RunAutoMacros` wykonuje się tylko po
   zaznaczeniu „Uruchom także makra otwarcia” (`:4687`, domyślnie wyłączone).

Wniosek z kolejności: jeśli proces zostanie zabity przed powrotem `New-Object` (krok 7), skoroszyt nie zostaje otwarty
i nie wykonuje się żadne makro. W incydencie błąd DCOM 10010 („serwer nie zarejestrował się”) pojawił się ok. 30 s po
każdym wykryciu. To pasuje do takiego scenariusza, ale nie jest dowodem.

Zastrzeżenie: po otwarciu skoroszytu `EnableEvents` wraca na `$true` jeszcze przed przeniesieniem edycji (`:4688-4690`).
Makra zdarzeń (`Worksheet_Change` itp.) mogą więc się wykonać bez zaznaczonej opcji, jeśli pozwala na to Centrum zaufania.

## 2. Dlaczego EDR uznaje to za atak

| Zachowanie | Miejsce | Typowa klasyfikacja (MITRE ATT&CK) |
| :--- | :--- | :--- |
| Ukryty `powershell.exe` uruchamiany z `pythonw.exe` | `:4861-4862` | T1059.001, T1564.003 |
| `-EncodedCommand` (UTF-16 base64) | `:4858`, `:4861` | T1027.010 |
| Drugi etap w base64 przez stdin → `ScriptBlock::Create` | `:4412`, `:4864` | T1027 (stager „pobierz i uruchom”) |
| `Add-Type` → `csc.exe` → DLL w pamięci | `:4652-4670` | T1027.004, T1106 |
| Office tworzony przez COM ze skryptu | `:4674` | T1559.001 |
| Wyliczanie procesów, `OpenProcess` z prawem `TERMINATE` | `:4342-4389` | T1057, T1106 |
| Odczyt i klikanie okien dialogowych innego procesu (`EnumWindows`, `WM_GETTEXT`, `BM_CLICK`, `SetForegroundWindow`) | `:6306-6475` | T1010 |

Każdy z tych elementów ma w kodzie uzasadnienie: stdin omija limit długości linii poleceń i politykę wykonywania skryptów,
`Add-Type` jest potrzebne do filtra komunikatów OLE. Mimo to razem dają dokładnie ten kształt, który EDR mają wykrywać.
Kodowanie base64 niczego też nie ukrywa: AMSI i log 4104 widzą odkodowany tekst. Zostaje wyłącznie wrażenie zaciemniania.

Ostatnie dwa wiersze tabeli działają dopiero po potwierdzeniu własnego procesu Excela. W incydencie najpewniej nie
zdążyły się wykonać, ale mają znaczenie przy przebudowie.

**Self-test.** `--self-test` zawiera testy działające tylko na Windows (`:18835-18920`, dołączane w `:20946`).
Uruchamiają one `powershell.exe -EncodedCommand`: jeden z bootstrapem przez stdin, jeden kompiluje `Add-Type`.
Na komputerze z EDR mogą wywołać podobny alarm.

## 3. Co zostaje na dysku po zabiciu procesu i jak odzyskać pracę

- **Projekt i arkusze Pivota** są zapisywane jako `%LOCALAPPDATA%\PivotStudio\recovery\<32 znaki hex>.json`. Zapis następuje
  przy każdej zmianie, ale tylko gdy projekt ma niezapisane zmiany (`:6494-6498`). Arkusz zapisuje się ok. 0,9 s po edycji
  (`:13118`). Plik znika tylko przy normalnym zamknięciu, więc po zabiciu procesu zostaje.
  „Plik → Odzyskaj kopię roboczą…” (`:15404`) pokazuje 5 najnowszych plików innych niż bieżący (`:6641-6644`).
  Przywraca sam dokument, bez ścieżki zapisu (`:6810`), dlatego potem trzeba użyć „Zapisz jako”.
- **Kopie skoroszytu z sesji Excela** leżą w `%LOCALAPPDATA%\PivotStudio\excel-sessions\session-*\`, a nie w `recovery`.
  Żaden ekran ich nie pokazuje, nic ich nie usuwa (`:5072-5073`), a „Odzyskaj kopię roboczą” ich nie obejmuje.
  Jeśli Excel nie zdążył otworzyć pliku, są to niezmienione kopie oryginału. Można to sprawdzić porównaniem SHA-256
  z `source_revision` w `session.json`. Oryginał nie jest nigdy zapisywany przez Pivot.
- `--data-dir` przenosi `recovery`, `sessions` i `excel-sessions`, ale nie przenosi `runtime\` ani `last-start-error.log`.

Strona Pythona reaguje bezpiecznie, ale ogólnikowo, gdy pomocniczy PowerShell zginie: odczyt dochodzi do EOF i pojawia się
komunikat „Proces obsługi Excel zakończył się nieoczekiwanie” (`:4866-4873`). Brakuje rozróżnienia „zablokowane przez
zabezpieczenia” od zwykłej awarii. Jeśli zabity zostanie sam proces GUI, nie ma czego obsłużyć.

## 4. Plan przebudowy mostu

### Docelowa architektura: osobny proces Pythona z comtypes

- Nowe ukryte polecenie `--excel-worker`, obsługiwane w `main()` tak jak `--worker` (`:24715-24717`).
  Funkcja `excel_worker_main()` powstaje na wzór `worker_main()` (`:5139`).
- Uruchamianie z `ExcelSessionController._run` tak jak w `WorkerClient` (`:5283-5289`): `[sys.executable, __file__, '--excel-worker']`,
  zamiana `pythonw.exe` na `python.exe` dla stdio, `CREATE_NO_WINDOW`. Znika `powershell.exe`, `-EncodedCommand`, base64,
  `ScriptBlock::Create` i `Add-Type`/`csc.exe`. Z pliku `.py` znikają też długie literały PowerShell i C#.
- **comtypes** (czysty Python) zamiast pywin32. Nie wymaga `pywin32_postinstall` ani rejestracji w systemie. Nowy Excel
  powstaje przez `comtypes.client.CreateObject('Excel.Application')` (serwer lokalny). O własności nadal decyduje
  zestaw PID sprzed startu + `Hwnd` + `ExcelOwnedProcess` (`:4365-4392`), bez zmian.
- **Filtr komunikatów OLE** do przeniesienia, nie do pominięcia. Bez niego wywołania do zajętego Excela kończą się od razu
  błędem `RPC_E_SERVERCALL_RETRYLATER`. Wariant: `IMessageFilter` jako `comtypes.COMObject`, rejestrowany przez
  `ctypes.OleDLL('ole32').CoRegisterMessageFilter`, z tą samą polityką co dziś
  (`rejectType == 2 && tickCount < 15000 ? 250 : -1`, `:4667`). Do potwierdzenia prototypem na Windows.
- Logika z `EXCEL_SESSION_POWERSHELL` przechodzi 1:1 do Pythona: `Read-Range`, `Apply-Edits` (z kontrolą oczekiwanych
  wartości), `Cell-State`, `Controls`/`Find-Control`, `Macro-Name`, `Session-Info`, limit 4 MB na komunikat po stronie
  workera (`:4481`). Protokół JSON-lines i nazwy zdarzeń (`busy/created/ready/done/error/fatal/closed`) zostają, więc
  `_send`/`_consume`/`submit` zmieniają się tylko w miejscu uruchomienia.
- Zależność: nowy profil `excel` z przypiętym `comtypes` w `REQUIREMENTS` (`:99`), `OPTIONAL_INSTALL_PROFILES` (`:5542`),
  `DEPENDENCY_MODULES`/`NAMES` (`:116`), `INSTALL_PROFILE_LABELS` (`:5540`) i pętli próbnych importów instalatora.
- `excel_availability()` przestaje wymagać PowerShella (`:4330-4332`) i sprawdza dostępność comtypes.
- `ExcelOwnedProcess`, `excel_session_foreground` i obsługa okien dialogowych (`:6306-6475`) to już czysty ctypes
  i zostają bez zmian. Warto ograniczyć `EnumWindows` do wątków procesu Excela (`EnumThreadWindows`).

### Wyłącznik i zachowanie awaryjne

- Nowe ustawienie `excel_bridge` (`auto`/`off`) na liście dozwolonych (`:6817`) i w oknie ustawień. Sprawdzają je
  `excel_availability()`, `start_session` (`:10632`) i `show_excel_session` (`:16364`).
- Przy wyłączonym moście: otwarcie prywatnej kopii w zwykłym Excelu użytkownika (`QDesktopServices.openUrl`) i ponowny
  import zapisanej kopii przez istniejącą ścieżkę `_excel_session_finished` (`:16390-16409`). Na żywo nie działa wtedy
  odczyt, przenoszenie edycji ani makra z Pivota, ale nie ma automatyzacji.
- Jednorazowa informacja przed pierwszym uruchomieniem, że zostanie uruchomiony proces pomocniczy sterujący Excelem
  i że zabezpieczenia firmowe mogą go zablokować.
- Komunikat po nagłej śmierci workera wskazujący możliwą blokadę przez zabezpieczenia i miejsce kopii roboczej.
- Testy Windows uruchamiające procesy pomocnicze w `--self-test` tylko po jawnym włączeniu.

### Testy do zmiany

Testy przywiązane do transportu PowerShell: `:18706` (`-EncodedCommand`, odkodowany bootstrap), `:18715-18716`,
`:18770-18772` (teksty w skrypcie PowerShell), `:18835`, `:18901`, `:18910` (uruchamiają `powershell.exe`).
Stub `fake_processor` (`:18636`) zostaje, ale linię `:18641` (odczyt base64 ze stdin) trzeba dopasować.
Testy zachowania kontrolera zostają. Zmieniają się też zdania o testach w `README.md`.

### Odrzucone warianty

- **PowerShell z pliku `.ps1` (`-File`)**: domyślna polityka wykonywania na klientach Windows blokuje niepodpisane skrypty.
  Obejście `-ExecutionPolicy Bypass` samo jest silnym sygnałem dla EDR, a `Add-Type` i tak zostaje. Ma sens tylko
  z podpisem firmowego certyfikatu.
- **VBScript / cscript / mshta**: gorsze od obecnego rozwiązania.
- **Tylko otwieranie w Excelu bez automatyzacji**: najbezpieczniejsze, ale traci większość funkcji. Dobre jako tryb awaryjny.

### Granice

- Python tworzący `Excel.Application` przez DCOM to nadal zachowanie, które EDR może punktować (zwykle słabiej niż
  ukryty PowerShell ze stagerem). Na komputerach zarządzanych przez IT trwałe rozwiązania to decyzja IT: wyjątek
  albo podpisany i dopuszczony program uruchamiający.
- Wyjątek po skrócie pliku nie przetrwa aktualizacji, bo plik zmienia się wiele razy dziennie.
- Projektu nie da się zweryfikować bez Windows z tym samym EDR. Każda próba na komputerze firmowym bez uzgodnienia
  z IT to nowy incydent.

## 5. Błąd: hasło Oracle trafia do pola „Schemat”

**Mechanizm (potwierdzony na prawdziwym oknie, offscreen).** Na stronie Oracle pola idą w kolejności: serwer, port,
service, SID, użytkownik, **schemat**, tryb (`:11059-11061`). Pole „Hasło” jest dodane pod całym stosem formularzy
(`:11077-11080`). Brakuje jakiegokolwiek `setTabOrder`. „Użytkownik → Tab” trafia więc do niemaskowanego pola
„Schemat (opcjonalny)”. Enter zapisuje połączenie z `schema='<hasło>'` i pustym hasłem. Ta sama pułapka jest na stronie
Firebird (Tab przechodzi do „Zestaw znaków” i „Rola”). H2 nie ma tego problemu.

Pierwsze połączenie kończy się błędem logowania, który kieruje uwagę na hasło, a nie na schemat. Po podaniu hasła
w osobnym oknie (`:15775-15777`) połączenie działa, a w pustym zakresie widać 0 obiektów. Nic nie ostrzega, że coś jest nie tak.

**Dokąd trafia wartość pola „Schemat” (jawnym tekstem):**

- plik projektu `.pivot` (SQLite z dokumentem JSON) oraz tabela `history` z 20 poprzednimi wersjami (`:1564-1573`).
  Poprawka i zapis w tym samym pliku nie usuwają starej wartości; czysty plik daje dopiero „Zapisz jako” pod nową ścieżką;
- `recovery\*.json` i `recovery\restarts\` (`:6496`, `:6862`);
- zakres katalogu w `cache\metadata-v1.sqlite` (`:3011`, `:3045`, `:3104-3110`), choć tożsamość cache schematu nie zawiera;
- okno zaufania źródła (`:15770` wypisuje wszystkie opcje), okno edycji, pole zakresu i statusy eksploratora („Schemat …”),
  eksport katalogu do JSON;
- serwer Oracle jako wartość bind `OWNER=:p1`. W trybie Thin bez TCPS połączenie zwykle nie jest szyfrowane;
- klucz w `SecretStore` (`:5397`) to skrót wszystkich opcji. Zmiana schematu osieroca zapisane hasło i powoduje, że
  „Puste = zachowaj hasło sesji” przestaje działać.

`safe_error` (`:241`) tej wartości nie maskuje.

**Poprawki w kodzie:**

1. Kolejność pól i Tab: „Hasło” zaraz po „Użytkowniku” na stronach Oracle i Firebird (`setTabOrder` w `switch_kind`
   albo pole hasła w każdym formularzu). Schemat i tryb przenieść niżej i dodać podpowiedź „Właściciel obiektów, np. HR”.
2. Sprawdzanie schematu w `SourceDialog.read` (`:11106-11127`), a nie w `validate_source`, które musi dalej wczytywać
   stare projekty. Odrzucać tylko wartości niemożliwe w Oracle: znak `"`, znaki sterujące, ponad 128 bajtów.
   Ostrzegać, gdy wartość nie spełnia reguły `^[A-Za-z][A-Za-z0-9_$#]*$` albo równa się hasłu. Dla nowego źródła
   ostrzegać też, gdy schemat jest wypełniony przy pustym haśle.
3. Przy otwieraniu projektu i odzyskiwaniu: ostrzeżenie o podejrzanym schemacie z propozycją wyczyszczenia i zapisu do
   nowego pliku. Do tego czasu bez automatycznej synchronizacji katalogu dla tego zakresu i z zamaskowaną wartością na ekranach.
4. Retencja: czyszczenie `history` przy zmianie opcji źródła, usuwanie slotów cache starego zakresu przy zmianie schematu,
   sprzątanie `recovery\restarts`.
5. Klucz `SecretStore` liczony z tożsamości połączenia dla danego typu źródła, bez schematu, trybu, zestawu znaków i roli,
   z migracją ze starego klucza.

Testy: test kolejności Tab dla Oracle i Firebird w testach okien natywnych (obok `:22820`), przypadki sprawdzania
schematu obok `:20772` oraz test, że zapis z czyszczeniem historii nie zostawia sekretu w pliku.

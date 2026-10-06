![Pivot Studio — od komórek do odpowiedzi. Zaznacz powiązane tabele, naciśnij Enter i otwórz wspólne dane.][hero]

<h1 align="center">Pivot Studio</h1>

<p align="center">
<b>Swoboda arkusza. Struktura bazy. Siła tabel przestawnych.</b><br>
Jedna lokalna aplikacja w pliku <code>PivotStudio.py</code>. Bez przeglądarki i bez rozbudowanej wstążki.
</p>

<p align="center">
<a href="https://github.com/guziczak/pivotstudio/raw/refs/heads/main/PivotStudio.py"><b>Pobierz PivotStudio.py ↓</b></a>
&nbsp; · &nbsp;
<a href="#szybki-start">Uruchom</a>
&nbsp; · &nbsp;
<a href="#zobacz-jak-pracujesz">Zobacz, jak pracujesz</a>
&nbsp; · &nbsp;
<a href="#status-i-ograniczenia">Status i ograniczenia</a>
</p>

---

Masz arkusz do uporządkowania? Bazę, której strukturę chcesz poznać? Dwie tabele, które dopiero razem odpowiadają na pytanie? **Pivot Studio łączy te zadania w jednym miejscu:** od komórek, przez mapę relacji i wspólne rekordy, po podsumowanie.

Zaczynasz od pliku. Narzędzia otwierasz wtedy, gdy ich potrzebujesz.

## Szybki start

Pobierz **[PivotStudio.py](https://github.com/guziczak/pivotstudio/raw/refs/heads/main/PivotStudio.py)** i uruchom:

```powershell
py PivotStudio.py
```

**Potrzebujesz Pythona 3.10–3.14, 64-bit, z Tcl/Tk.** Pierwszy start otwiera wybór **publicznego PyPI** albo **własnego Artifactory**, a następnie przygotowuje prywatne środowisko bibliotek. Domyślnie zaznaczone jest także przygotowanie pakietów sterowników baz danych; wybór można zmienić przed pobraniem. Nie musisz ręcznie wykonywać poleceń `pip`.

Od razu możesz upuścić XLSX, XLSM lub SQLite na okno. XLSM otwiera dane bez uruchamiania makr. Bez własnych danych: **menu → Pomoc → Otwórz przykład**.

<details>
<summary><b>Wymagania, inne systemy i opcjonalne sterowniki</b></summary>

Windows jest główną platformą docelową. Na Linux/macOS uruchomienie to `python3 PivotStudio.py`; dostępność bibliotek zależy od platformy, a pełny odbiór tych systemów pozostaje otwarty.

Dla wszystkich opcjonalnych sterowników potrzebny jest **Python 3.11 lub nowszy** — przypięty sterownik Firebirda nie obsługuje Pythona 3.10.

| Źródło | Dodatkowe składniki |
| :--- | :--- |
| **SQLite** | Obsługa w Pythonie; bez osobnego serwera. |
| **H2** | JPype1, zgodna 64-bitowa Java i wskazany JAR H2 2.x. |
| **Firebird** | `firebird-driver` i zgodna biblioteka klienta Firebird. |
| **Oracle** | `python-oracledb`; w trybie Thick także Oracle Client. |

Biblioteki i interpreter są osobnymi składnikami. Jeden plik `.py` oznacza prostą dystrybucję kodu aplikacji, nie samodzielny plik EXE.

</details>

## Zobacz, jak pracujesz

### 01 · Otwórz bazę, nie formularz

Przeciągnij plik SQLite na okno. Dostajesz **katalog tabel, widoków, kolumn, kluczy i indeksów** — bez obowiązkowego tworzenia analizy. Przełącz się na **Relacje**, żeby zobaczyć, co z czym się łączy.

Powiązane tabele są blisko siebie. Linie prowadzą między polami kluczy. **Widok → Uporządkuj według relacji** wykorzystuje aktualną szerokość kanwy: liczba kart w rzędzie zależy od miejsca, bez stałego ograniczenia do czterech. Polecenie przelicza układ, wraca do jego początku i przywraca czytelne powiększenie 100%. Mapę możesz przesuwać i powiększać; ręczny układ pozostaje po powrocie z danych oraz podczas doczytywania kolumn, do następnego jawnego uporządkowania.

Duży katalog Oracle jest wyświetlany **stronami po 1000 obiektów**. Możesz przeglądać wszystkie dostępne schematy albo jeden duży schemat; liczba ponad 20 000 obiektów nie blokuje otwarcia listy. Pivot zapisuje listę obiektów i odczytane szczegóły w lokalnym **cache SQLite**, zachowanym po zamknięciu aplikacji. Po ukończeniu synchronizacji wyszukiwanie nazw i przechodzenie między stronami korzystają z zapisanej listy. Przy braku kompletnego cache dostępny pozostaje odczyt stron z Oracle.

Cache zawiera **metadane, nie rekordy tabel**. Po zapisaniu listy obiektów Oracle aplikacja stopniowo pobiera **kolumny i klucze, w tym relacje FK, dla całego wybranego zakresu**. Odczyt obejmuje małe partie do 20 obiektów i ustępuje pierwszeństwa bieżącej pracy z bazą. Postęp zapisuje się w SQLite; można wstrzymać pobieranie i wznowić je później, również po ponownym uruchomieniu aplikacji. Błędy i częściowo odczytane obiekty mają osobny status. Indeksy, wyrażenia i definicje nadal są doczytywane po wybraniu obiektu.

Kolumny i połączenia pojawiają się na mapie w trakcie pobierania, bez resetowania powiększenia i ręcznie przesuniętych kafelków. Widok otoczenia tabeli wyszukuje relacje wychodzące i przychodzące w całym zapisanym katalogu, także poza bieżącą stroną. Pełna lista nazw nie oznacza jeszcze pełnej struktury; brakujące metadane nie są przedstawiane jako potwierdzone połączenia. Zapisany katalog można przeglądać bez sieci i sterownika. Odczyt struktury w tle, aktualne dane oraz wykonanie zapytań wymagają dostępu do bazy; otwarcie projektu nie uruchamia zdalnego odczytu bez potwierdzenia źródła.

Pole **Schemat** w połączeniu ustala domyślny zakres. Możesz go tymczasowo zmienić w **Zakres Oracle → Odczytaj schemat**, bez zmiany poświadczeń; nazwę schematu można także wpisać ręcznie. Obiekty systemowe można włączyć polem **Systemowe**. Odświeżenie przygotowuje nową listę w tle; anulowanie lub błąd zachowuje poprzednią kompletną wersję. Cache jest oddzielony według połączenia, konta i zakresu, znajduje się poza projektem i można go wyczyścić z menu **Baza**. Na Windows jest to plik `%LOCALAPPDATA%\PivotStudio\cache\metadata-v1.sqlite` (przy własnym `--data-dir` — podkatalog `cache` wybranego katalogu). Zapis zawiera nazwy i odczytane definicje, bez haseł. Mapa pokazuje rozpoznane dotąd relacje, a eksport JSON obejmuje bieżącą stronę i doczytane szczegóły. Ograniczenia pamięci i rozmiaru pojedynczej odpowiedzi pozostają aktywne.

![Mapa struktury bazy: zaznaczone events i operations, wyróżnione pola klucza i polecenie otwarcia połączonych danych.][relations]

W zakładce **Dane** dwuklik komórki doczytuje jej odroczoną zawartość LOB/BLOB lub długi tekst. Okno pokazuje tekst, JSON albo podgląd szesnastkowy danych binarnych. Rozmiar i ewentualne skrócenie są jawne; **Zapisz całość…** pobiera pełną wartość do pliku strumieniowo. Odczyt wskazuje konkretny rekord przez klucz lub obsługiwany identyfikator wiersza, także w wyniku JOIN. Widok bez jednoznacznego identyfikatora otrzymuje komunikat zamiast próby ponownego odczytu według numeru pozycji. Wartość jest odczytywana na bieżąco, więc może różnić się od wcześniejszego podglądu strony.

### 02 · Zaznacz. Enter. Wspólne dane.

Zaznacz powiązane tabele przez **Ctrl+klik** lub prostokąt i naciśnij **Enter**. Pivot otworzy wspólną kartę według zadeklarowanej relacji — bez przepisywania kluczy i bez ręcznego pisania SQL.

Na przykład `events.operation_id → operations.operation_id`: zdarzenie i przypisana do niego operacja w jednym wierszu. Nagłówki wskazują pochodzenie pól, a rodzaj połączenia można zmienić między **wszystkimi rekordami podstawy (LEFT)** i **tylko dopasowanymi (INNER)**. Gdy relacja jest niejednoznaczna, wybierasz ją jawnie.

![Wynik połączenia events i operations: wspólna tabela z nazwami źródeł w nagłówkach, filtrem i kartą wyniku.][joined]

<sub>Identyfikatory operacji, zatwierdzeń i znaczniki czasu na zdjęciu zamaskowano.</sub>

### 03 · Z rekordów zrób odpowiedź

**Region do wierszy. Miesiąc do kolumn. Kwota do wartości.** Tak powstaje podsumowanie, które można filtrować, zapisać i wyeksportować. Dostępne są między innymi sumy, średnie, liczba rekordów i liczba wartości unikalnych.

Przykład: **jak rozkłada się sprzedaż między regionami?**

| Region | 2026-01 | 2026-02 | Ogółem |
| :--- | ---: | ---: | ---: |
| Południe | 9 000 | 13 500 | 22 500 |
| Północ | 12 000 | 15 000 | 27 000 |
| Zachód | 7 500 | 11 000 | 18 500 |
| **Ogółem** | **28 500** | **39 500** | **68 000** |

<sub>Fikcyjne dane: wynik obliczony silnikiem Pivot Studio 0.7.2 dla 12 rekordów. To tabela w README, nie zrzut interfejsu.</sub>

Zaznaczenie w arkuszu może stać się źródłem pivota lub wykresu. Połączenie tabel może zasilić analizę pełnego wyniku, a nie tylko aktualnie oglądanej strony. **Przy relacji jeden-do-wielu dobierz właściwy poziom agregacji:** wartość z tabeli nadrzędnej może wystąpić w kilku wierszach JOIN-a.

### 04 · Kiedy potrzebujesz arkusza — masz arkusz

**A, B, C… u góry. 1, 2, 3… z lewej.** Aktywna komórka, pasek formuły, zakresy, zakładki i zoom. Do tego edycja, kopiowanie i wklejanie, cofanie, podstawowe formatowanie i lokalny silnik formuł.

XLSX i XLSM otwierasz jako skoroszyt z nazwanymi arkuszami. Przełączasz zakładki bez ponownego importowania pliku. Narzędzia **Format**, **Dane** i **Pivot** są pod ręką, ale nie zajmują czterech rzędów ekranu.

**Otwarcie XLSM w arkuszu Pivot odczytuje dane bez wykonywania VBA.** Początkowy widok korzysta z wyników formuł zapisanych w pliku i wskazuje ich pochodzenie. Po lokalnej zmianie zapisane wyniki tracą ważność: obsługiwane formuły przelicza Pivot, pozostałe wymagają Excela. Ukryte wiersze i kolumny, kolory dokumentu i rozpoznane przyciski arkusza są zachowywane w podglądzie. Formatowanie warunkowe i wynik działania makr można odczytać w sesji Excel. Oryginał pozostaje bez zmian; eksport lokalnego modelu do XLSX wymaga akceptacji utraty nieobsługiwanych elementów, w tym makr.

**Narzędzia → Excel: makra i PDF** oraz przyciski zgodności w arkuszu otwierają sesję wymagającą Microsoft Excel na Windows. Excel posiada rzeczywisty skoroszyt i wykonuje obliczenia oraz VBA. Pivot pokazuje wybrany zakres z wynikami, formułami, ukryciem, scaleniami i kolorami odczytanymi z Excela, w tym formatowaniem warunkowym. Edycja w tym widoku trafia do tej samej sesji; przy konflikcie z nowszą zawartością komórki zapis jest zatrzymywany. Widok jest odświeżany jawnie oraz po poleceniach Pivot; po bezpośredniej pracy w oknie Excela wybierz odświeżenie.

Przed uruchomieniem sesji z lokalnego arkusza Pivot sprawdza wersję źródłowego pliku i przygotowuje wyłącznie zmienione wartości i formuły (do 1000 komórek). Zmiany struktury lub formatowania wykonuj w sesji Excel — nie są po cichu pomijane przy przenoszeniu. Podczas sesji uruchomionej z arkusza lokalna edycja jest zablokowana. Zapisana kopia może następnie odświeżyć ten sam arkusz projektu; wynik z innego projektu ani spóźniona odpowiedź nie nadpisują bieżącej pracy.

Rozpoznane kontrolki mają podpisy i pozycje w lokalnym podglądzie. Polecenie pokazania kontrolki przechodzi do niej w prawdziwym Excelu. **Uruchomienie przypisanego makra** jest osobną akcją dla obsługiwanych odwołań `OnAction`; nie odtwarza `Application.Caller` ani zdarzeń ActiveX. Przy takich zależnościach użyj rzeczywistego przycisku w Excelu. Nie są emulowane wszystkie rysunki, kontrolki ActiveX ani niestandardowe wstążki.

Opcja uruchomienia makr otwarcia pozwala wykonać inicjalizację skoroszytu. Nie łączy się jej z początkowym przeniesieniem lokalnych edycji, ponieważ `Workbook_Open` wykonałoby się przed nimi. Pivot nie potrzebuje kodu VBA i nie zmienia ustawień Centrum zaufania; zachowuje oznaczenie pochodzenia kopiowanego pliku. Polecenia zmieniające skoroszyt i makra nie są automatycznie ponawiane po błędzie. Zdarzenia arkusza mogą zmienić inne komórki lub wywołać działania poza plikiem; częściowo zastosowana operacja pozostaje jawna i wymaga sprawdzenia w tej samej sesji.

Pliki towarzyszące, np. szablon Worda, można wskazać przy otwieraniu sesji. Wybrane pliki są kopiowane obok skoroszytu z zachowaniem nazw; Pivot nie odgaduje wszystkich zależności VBA ani nie zmienia ścieżek zapisanych w makrze. Makro może uruchomić zainstalowanego Worda, ale jego okna i zapisy obsługuje sam Office. Pivot nie przejmuje ani nie zamyka cudzych procesów Worda.

Standardowe okna komunikatów należące do tej sesji pojawiają się w ramce Pivot wraz z rzeczywistymi odpowiedziami. **„Pomiń → Nie”** przy pytaniu „Czy przerwać sprawdzanie?” wysyła właśnie odpowiedź „Nie”. To nie zmienia warunków w kodzie VBA i nie gwarantuje, że skrypt mimo błędu wygeneruje raport. Niestandardowe formularze obsługujesz w widocznym Excelu. Odpowiedź jest przekazywana wyłącznie po kliknięciu, po ponownym sprawdzeniu aktualnego okna i procesu.

**PDF bieżącego arkusza** korzysta z drukowania Excela, niezależnie od przycisku generatora w skoroszycie. Zapisuje aktualną zawartość i obszar wydruku — nie potwierdza wykonania obliczeń generatora. **Zapisz kopię skoroszytu** zachowuje format, w tym VBA w XLSM. Można również zachować stan sesji w jej katalogu. Operacje potwierdzają sukces po sprawdzeniu pliku wynikowego.

Katalogi sesji w `%LOCALAPPDATA%\PivotStudio\excel-sessions` (lub pod własnym `--data-dir`) pozostają po zamknięciu aplikacji; dotyczy to również zapisanych tam dokumentów utworzonych przez makra. W oknie sesji można otworzyć ten folder. Przed zamknięciem zapisz zmiany komórek — zachowanie katalogu nie oznacza zapisania niezapisanej pamięci Excela. Pliki utworzone przez makro w innych miejscach pozostają pod kontrolą tego makra.

![Lokalny arkusz Pivot Studio: kompaktowy nagłówek, pasek formuły, oznaczenia komórek i zakładki na dole.][sheet]

<details>
<summary><b>Praca z klawiaturą</b></summary>

| Klawisz | W arkuszu |
| :--- | :--- |
| **Enter / Shift+Enter** | Zatwierdź wpis i przejdź w dół / w górę. |
| **Tab / Shift+Tab** | Zatwierdź wpis i przejdź w prawo / w lewo. |
| **Escape** | Anuluj niezapisaną edycję komórki. |
| **F2** | Edytuj aktywną komórkę. |
| **Ctrl+C / Ctrl+V** | Kopiuj / wklej zakres. |
| **Ctrl+K** | Wyszukaj polecenie. |

Obsługa zatwierdzania i nawigacji została przebudowana w **0.7.2**. Stan jej weryfikacji jest opisany w sekcji [Status i ograniczenia](#status-i-ograniczenia). Enter na diagramie ma osobne działanie: otwiera zaznaczone dane.

</details>

## Jedno przygotowanie. Twoje źródło bibliotek.

Na pierwszym ekranie wybierasz, skąd pobrać pakiety. **Własny adres Artifactory**, opcjonalny login i hasło/token; dla sieci firmowej także CA i proxy. Jest również możliwość wskazania kompletu pakietów offline.

<p align="center">
<img src="docs/screenshots/setup-artifactory.png" width="720" alt="Okno przygotowania Pivot Studio z wyborem własnego Artifactory, adresem indeksu i opcjonalnym logowaniem.">
</p>

Hasło źródła pakietów nie jest zapisywane. Błąd firmowego indeksu **nie przełącza pobierania samowolnie na PyPI**. Przygotowane biblioteki pozostają w prywatnym środowisku — nie trzeba instalować ich globalnie.

Od **0.7.4** brak sterownika przy połączeniu pokazuje trwały komunikat z przyciskiem **Przygotuj sterownik**. Uszkodzony import ma osobną akcję naprawy; brak klienta natywnego prowadzi do ustawień. To samo okno przygotowania przywraca ostatnie źródło, adres, login i ustawienia sieci. Hasło lub token Artifactory podajesz ponownie; zmiana adresu usuwa wpisany sekret.

Pierwsze przygotowanie sprawdza zestaw podstawowy, następnie wybrane dodatki: Oracle, Firebird, H2/JPype i magazyn poświadczeń. Brak dodatku pozwala jawnie wybrać ponowienie lub uruchomienie sprawdzonego zestawu. Java, JAR H2, fbclient i Oracle Client pozostają oddzielnymi składnikami. Pakiet Oracle w trybie Thin nie wymaga Oracle Client.

Po doinstalowaniu wybierasz **Zapisz i uruchom ponownie** albo **Później**. Pivot zapisuje i ponownie odczytuje prywatną kopię pracy; stare okno zamyka dopiero po potwierdzeniu startu nowego. Kopia nie nadpisuje dotychczasowego pliku projektu: zachowuje jego nazwę i stan niezapisanych zmian. Hasła i zgody na dostęp do bazy nie przechodzą do nowej sesji — przywrócone źródło czeka na **Połącz**.

## Lokalnie, z jasnymi zasadami

**Arkusz edytujesz. Bazę przeglądasz.** Połączenia bazodanowe i ich wyniki pozostają tylko do odczytu. Polecenie **„Kopia strony → arkusz”** tworzy edytowalną kopię bieżącej strony. Eksport XLSX zapisuje nową kopię skoroszytu, nie nadpisuje otwartego oryginału.

**Zapisujesz pracę, nie tylko obraz tabeli.** Projekt `.pivot` zachowuje arkusze, analizy i nazwane plany połączeń. Przygotowane dane można dołączyć do projektu; oryginalny XLSX/XLSM jest osobną decyzją. Nie jest to automatyczna kopia całego serwera bazodanowego.

**Wiesz, czego używasz.** W **Pomoc → Technologie i licencje** sprawdzisz lokalne wersje składników, deklaracje i warunki, źródła oraz dostępne teksty licencji. Zestawienie można skopiować lub wyeksportować; nieznane wersje wymagają weryfikacji, nie dostają automatycznie zielonego „tak”.

Aplikacja nie ma wbudowanej telemetrii ani WebEngine. Sieć jest potrzebna przy wybranym pobieraniu pakietów, połączeniach z serwerami baz i otwieraniu źródeł dokumentacji. **Projekty, importy i migawki nie są szyfrowane.**

## Status i ograniczenia

**0.7.4 · aktywnie rozwijane wydanie.** Nowe przygotowanie sterowników, kontrolowany restart oraz aktualny złoty znak Przesmyk w **Pomoc → O autorze**. Poprawiono zapis projektu, eksport i tworzenie pivota z zakresu na Windows, a także filtrowanie i sortowanie `DECIMAL_TEXT`, szczegóły pivota dla dat SQLite oraz formatowanie zakresów ze scaleniami. Pivot Studio nie jest pełnym zamiennikiem Excela. Obsługa formuł i formatowania ma określony zakres; lokalny arkusz odczytuje dane XLSM, a wykonanie VBA wymaga osobnej sesji z zainstalowanym Excelem. Część obiektów skoroszytów nie jest obsługiwana przez lokalny edytor. Przy ważnych plikach zachowaj oryginał i sprawdź eksportowaną kopię.

Roboczy skoroszyt ma limit **200 000 zapisanych komórek**, a pojedyncza operacja — **100 000**. Plan JOIN obejmuje **2–8 różnych tabel z jednego źródła**. Płaskie połączenie nie jest wielotabelowym modelem miar Excela. Widoczność katalogu bazy zależy od uprawnień konta.

<details>
<summary><b>Stan testów i pochodzenie materiałów</b></summary>

Weryfikacja **0.7.4 na Windows, Python 3.12 i 3.14**: na każdej wersji ostatni pełny przebieg obejmował **614 testów rdzenia — 612 przeszło, 2 pominięto z powodu braku uprawnienia do dowiązań**. Regresje obejmują zapis i eksport, pivot z zakresu, liczby, daty, scalenia oraz katalog **49 295 obiektów**. Sprawdzono strumieniowy zapis katalogu przez protokół workera, ponowne otwarcie SQLite, ostatnią stronę, lokalne wyszukiwanie, szczegóły oraz JOIN. Zestawy cache obejmują 21 testów magazynu, 20 testów serwisu, 12 testów pobierania struktury i 11 testów mapy: anulowanie, wznowienie po restarcie, pierwszeństwo zadań użytkownika, ponowne użycie połączenia z nową transakcją dla każdej partii, zmiany bazy i generacji, limity pamięci i dysku, błędy częściowe, relacje spoza strony oraz odbudowę uszkodzonego cache. Zapytania Oracle wykonano na lokalnym słowniku testowym, bez serwera Oracle.

Nowe sprawdzenia obejmują 23 testy odczytu i eksportu pojedynczej komórki, 3 testy ich obsługi przez serwis, 6 testów danych XLSM oraz 2 testy układu mapy. Zweryfikowano tożsamość wiersza i kolumny, klucze złożone oraz binarne, anulowanie eksportu bez utraty poprzedniego pliku, ograniczenie podglądu, dokładne liczby JSON i dekodowanie Unicode na granicach porcji. Testy SQLite używają rzeczywistej bazy; zachowanie LOB-ów Oracle, Firebird i H2 sprawdzono na kontrolowanych odpowiednikach sterowników. Kontener XLSM zachowuje oryginalne bajty, a samo otwarcie danych nie wykonuje VBA.

Obsługę zewnętrznego Excela obejmują **26 testów kontrolera sesji, 12 testów przekazywania zmian, 9 testów zgodności importu Office i 11 testów komunikatów Win32**. Sprawdzono osobny proces, trwałość wyników, zapis kopii, pliki towarzyszące, identyfikację skoroszytu/arkusza/kontrolki, konflikty komórek, częściowo wykonane edycje bez ponawiania, źródłowe SHA256 i pochodzenie wyników formuł. Standardowe okna Windows sprawdzono w rzeczywistym osobnym procesie. Funkcje skryptu Windows PowerShell wykonano na kontrolowanych obiektach testowych: odczyt zakresu i formatowania, literalny tekst, formuły, błędy komórek, ukrycie, scalenia, kontrolki oraz odrzucenie usuniętego arkusza zastąpionego nowym o tej samej nazwie. Transport poleceń testowano przez proces zastępczy. Na stanowisku testowym **nie ma zarejestrowanego Excel COM**; wykonania VBA ani eksportu przez prawdziwego Excela nie potwierdzono. Sprawdzono rzeczywisty start pomocnika i kontrolowane zakończenie przy braku COM.

Zestaw przygotowania obejmuje **70 wcześniej sprawdzonych testów** (pełny przebieg 69 oraz dodatkowa regresja i ponowny przebieg logiki instalatora po poprawce). Rzeczywista instalacja z publicznego PyPI w osobnym katalogu przygotowała wszystkie pięć profili, przeszła `pip check` i importy. Sprawdzono także rzeczywisty restart Qt z kopią niezapisanej pracy i potwierdzeniem startu nowego okna.

Ostatni pełny przebieg Qt w trybie `offscreen` objął **233 testy: 228 przeszło, 5 nie**. Te pięć niepowodzeń — skróty klawiaturowe, IME, układ nagłówka i limit czasu wykazu technologii — odtworzono także na kodzie sprzed zmian Office (`a1e322a`). Po podsumowaniu pełnego przebiegu wystąpiła awaria procesu przy wyjściu (`0xC0000005`); dziennik Windows zawiera identyczną sygnaturę także sprzed tej zmiany, ale przyczyna wymaga osobnej diagnozy. Nie jest to w pełni zaliczony test całego GUI. Połączenie z firmowym Artifactory i rzeczywistymi instancjami H2, Firebird oraz Oracle wymaga dostępu do tych systemów.

Katalog Oracle, trwały cache i powiązane funkcje sprawdzono w **84 testach Qt — wszystkie przeszły**. Oprócz wcześniejszych 70 scenariuszy zestaw obejmuje 11 testów okna pełnej komórki, 2 testy otwierania XLSM i test rozmieszczania tabel po zmianie szerokości okna. Dwa testy przechodzą cały przepływ SQLite → worker → okno komórki → eksport pełnej wartości. Sprawdzono autoryzację odczytu w tle, pauzę, nieaktualne odpowiedzi, zachowanie kolumn sąsiada po odczycie szczegółów, błędy częściowe i zmianę generacji. „Uporządkuj” zmienia liczbę kart w rzędzie wraz z dostępną szerokością; wzrost kart po pobraniu kolumn nie powoduje kolizji z automatycznie rozmieszczonymi sąsiadami i zachowuje ręczne pozycje, zaznaczenie oraz powiększenie. Rzeczywisty widok Qt odczytał 49 295 nazw z pliku SQLite bez połączenia z Oracle. Interfejs nie modyfikuje odpowiedzi równolegle zapisywanej do cache. Scenariusze interfejsu Oracle korzystają z danych testowych.

Bieżącą obsługę Office sprawdzono dodatkowo w **33 testach Qt — wszystkie przeszły**, po końcowych poprawkach układu okna. Obejmują kontrolki na arkuszu, źródłowe kolory i ukrycie, zapisane wyniki formuł, native snapshoty i jawne edycje, wybór arkusza, nieaktualne odpowiedzi, częściowe błędy, zapis przed zamknięciem i powrót do właściwego projektu. Osobne zakładki arkusza i komunikatów mieszczą się w oknie 900×760; sprawdzono również rzeczywiste renderowanie na syntetycznych danych. Kontroler Excel jest w tych testach zastępowany kontrolowanym odpowiednikiem, a zrzuty fixture nie stanowią dowodu wykonania VBA w Office.

Instalator sprawdza hashe pobranych plików i ponownie używa wyłącznie zweryfikowanych plików z tego samego źródła. Nie zawiera jeszcze kompletnego, wcześniej zatwierdzonego manifestu hashy wydawców dla wszystkich platform.

Zdjęcia przedstawiają rzeczywiście uruchomioną aplikację, nie makiety. Mapa i wynik JOIN-a pochodzą z Windows, z wersji 0.7.0; arkusz — z udostępnionego zdjęcia sprzed poprawki 0.7.2. Okno przygotowania to Tkinter 0.7.1 na Linux/Xvfb. Baner wykorzystuje fragment autentycznego diagramu. Zrzuty przycięto bez paska zadań; w wyniku JOIN-a zamaskowano techniczne identyfikatory i znaczniki czasu. Nie są to nowe zrzuty GUI 0.7.2.

Mała tabela sprzedaży powyżej powstała z fikcyjnego, dwumiesięcznego zbioru. Jej wartości i sumy obliczono przez `run_pivot()` z niezmienionego kodu 0.7.2 i niezależnie porównano z agregacją SQLite. Nie przedstawia danych użytkownika ani nagrania GUI.

Wbudowane sprawdzenia, uruchamiane jawnie:

```powershell
py PivotStudio.py --self-test       # rdzeń, bez Qt i bez sieci
py PivotStudio.py --bootstrap-test  # przygotowanie i Tkinter, bez pobierania
py PivotStudio.py --ui-test         # prawdziwe kontrolki; wymaga PySide6
```

</details>

## Licencja

Własny kod aplikacji jest dostępny na **[MIT](LICENSE)**. Biblioteki, Java i klienci baz mają odrębne licencje oraz warunki dystrybucji. Wykaz w aplikacji pomaga je sprawdzić, ale nie zastępuje audytu konkretnego zestawu komponentów.

Do przekazania programu wystarczy **`PivotStudio.py`**. W repozytorium są tylko kod, ten README, licencja i obrazy w `docs/screenshots/` — bez dołączonych bibliotek, venv i prywatnych plików baz.

---

<p align="center"><b>Otwórz dane. Zobacz powiązania. Znajdź odpowiedź.</b></p>
<p align="center"><a href="https://github.com/guziczak/pivotstudio/raw/refs/heads/main/PivotStudio.py">Pobierz Pivot Studio</a> · <a href="#szybki-start">Wróć do startu ↑</a></p>

[hero]: docs/screenshots/hero.png
[relations]: docs/screenshots/database-relations.png
[joined]: docs/screenshots/joined-data.png
[sheet]: docs/screenshots/spreadsheet.png

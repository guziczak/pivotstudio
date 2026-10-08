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

Przycisk **Uporządkuj** grupuje tabele według ich powiązań i skraca połączenia w obu kierunkach mapy. Tabele z wieloma relacjami otacza sąsiadami, a gęsto powiązane grupy trzyma blisko siebie. Układ uwzględnia rzeczywiste rozmiary kart i zostawia między nimi przejścia dla strzałek; może rozrosnąć się poza ekran. Strzałki prowadzą między polami kluczy, omijają pozostałe tabele i wybierają osobne trasy, aby ograniczyć nakładanie oraz skrzyżowania. Trasy są przeliczane także po ręcznym przesunięciu karty. Polecenie przywraca czytelne powiększenie 100%; **Widok → Dopasuj całą mapę** pokazuje całość. Ręczny układ pozostaje po powrocie z danych oraz podczas doczytywania kolumn, do następnego jawnego uporządkowania.

Duży katalog Oracle jest wyświetlany **stronami po 1000 obiektów**. Możesz przeglądać wszystkie dostępne schematy albo jeden duży schemat; liczba ponad 20 000 obiektów nie blokuje otwarcia listy. Pivot zapisuje listę obiektów i odczytane szczegóły w lokalnym **cache SQLite**, zachowanym po zamknięciu aplikacji. Po ukończeniu synchronizacji wyszukiwanie nazw i przechodzenie między stronami korzystają z zapisanej listy. Przy braku kompletnego cache dostępny pozostaje odczyt stron z Oracle.

Cache zawiera **metadane, nie rekordy tabel**. Po zapisaniu listy obiektów Oracle aplikacja stopniowo pobiera **kolumny i klucze, w tym relacje FK, dla całego wybranego zakresu**. Odczyt obejmuje małe partie do 20 obiektów i ustępuje pierwszeństwa bieżącej pracy z bazą. Postęp zapisuje się w SQLite; można wstrzymać pobieranie i wznowić je później, również po ponownym uruchomieniu aplikacji. Błędy i częściowo odczytane obiekty mają osobny status. Indeksy, wyrażenia i definicje nadal są doczytywane po wybraniu obiektu.

Kolumny i połączenia pojawiają się na mapie w trakcie pobierania, bez resetowania powiększenia i ręcznie przesuniętych kafelków. Widok otoczenia tabeli wyszukuje relacje wychodzące i przychodzące w całym zapisanym katalogu, także poza bieżącą stroną. Pełna lista nazw nie oznacza jeszcze pełnej struktury; brakujące metadane nie są przedstawiane jako potwierdzone połączenia. Zapisany katalog można przeglądać bez sieci i sterownika. Odczyt struktury w tle, aktualne dane oraz wykonanie zapytań wymagają dostępu do bazy; otwarcie projektu nie uruchamia zdalnego odczytu bez potwierdzenia źródła.

Pole **Schemat** w połączeniu ustala domyślny zakres. Możesz go tymczasowo zmienić w **Zakres Oracle → Odczytaj schemat**, bez zmiany poświadczeń; nazwę schematu można także wpisać ręcznie. Obiekty systemowe można włączyć polem **Systemowe**. Odświeżenie przygotowuje nową listę w tle; anulowanie lub błąd zachowuje poprzednią kompletną wersję. Cache jest oddzielony według połączenia, konta i zakresu, znajduje się poza projektem i można go wyczyścić z menu **Baza**. Na Windows jest to plik `%LOCALAPPDATA%\PivotStudio\cache\metadata-v1.sqlite` (przy własnym `--data-dir` — podkatalog `cache` wybranego katalogu). Zapis zawiera nazwy i odczytane definicje, bez haseł. Mapa pokazuje rozpoznane dotąd relacje, a eksport JSON obejmuje bieżącą stronę i doczytane szczegóły. Ograniczenia pamięci i rozmiaru pojedynczej odpowiedzi pozostają aktywne.

![Mapa struktury bazy: zaznaczone events i operations, wyróżnione pola klucza i polecenie otwarcia połączonych danych.][relations]

W zakładce **Dane** dwuklik komórki doczytuje jej odroczoną zawartość LOB/BLOB lub długi tekst. Okno pokazuje tekst, JSON albo podgląd szesnastkowy danych binarnych. Rozmiar i ewentualne skrócenie są jawne; **Zapisz całość…** pobiera pełną wartość do pliku strumieniowo. Odczyt wskazuje konkretny rekord przez klucz lub obsługiwany identyfikator wiersza, także w wyniku JOIN. Widok bez jednoznacznego identyfikatora otrzymuje komunikat zamiast próby ponownego odczytu według numeru pozycji. Wartość jest odczytywana na bieżąco, więc może różnić się od wcześniejszego podglądu strony.

### 02 · Zaznacz. Enter. Wspólne dane.

Zaznacz powiązane tabele przez **Ctrl+klik** lub prostokąt i naciśnij **Enter**. Pivot otworzy wspólną kartę według zadeklarowanej relacji — bez przepisywania kluczy i bez ręcznego pisania SQL.

Kliknięcie tabeli wyróżnia wszystkie wchodzące i wychodzące strzałki, a pozostałe przygasza. Przy zaznaczeniu kilku tabel wyróżnione są połączenia każdej z nich. Kliknięcie pustego miejsca przywraca zwykły widok mapy.

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

**Narzędzia → Excel: makra i PDF** oraz przyciski zgodności w arkuszu otwierają sesję wymagającą Microsoft Excel na Windows. Excel posiada rzeczywisty skoroszyt i wykonuje obliczenia oraz VBA. Pivot pokazuje wybrany zakres z wynikami, formułami, ukryciem, scaleniami i kolorami odczytanymi z Excela, w tym formatowaniem warunkowym. Edycja w tym widoku trafia do tej samej sesji; przy konflikcie z nowszą zawartością komórki zapis jest zatrzymywany. Podgląd odświeża się co kilka sekund tylko wtedy, gdy okno Pivot jest aktywne, a zakładka **Arkusz Excela** widoczna; nie odczytuje Excela, gdy ten czeka na odpowiedź. **Odczytaj** odświeża go od razu.

Okno sesji mieści się w obszarze roboczym ekranu, także przy skalowaniu 125–150%: zakładki przewijają się, a stopka pozostaje widoczna. W stopce jest **Uruchom sesję Excela**, a po otwarciu skoroszytu **Uruchom** dla przycisku wybranego w polu **Przycisk** oraz **Pokaż w Excelu**. Excel zaczyna pracę z ukrytym głównym oknem; pokazanie go wymaga jawnej akcji. Jedyny przycisk arkusza jest wybierany od razu; przy kilku trzeba wskazać jeden. Kliknięcie w trakcie odczytu podglądu czeka na jego koniec i wykonuje się raz. Pełny błąd pozostaje w zakładce **Komunikaty**, a szczegóły można otworzyć i skopiować.

Przed uruchomieniem nowej sesji z lokalnego arkusza Pivot sprawdza wersję źródłowego pliku i przygotowuje zmienione wartości i formuły (do 1000 komórek) oraz szerokości kolumn i wysokości wierszy. Sama zmiana wymiarów zachowuje zapisane wyniki formuł i ma Cofnij/Ponów. Pozostałe zmiany struktury lub formatowania wykonuj w sesji Excel — nie są po cichu pomijane przy przenoszeniu. Podczas sesji uruchomionej z arkusza lokalna edycja jest zablokowana. Zapisana kopia może następnie odświeżyć ten sam arkusz projektu; wynik z innego projektu ani spóźniona odpowiedź nie nadpisują bieżącej pracy.

Rozpoznane kontrolki mają podpisy i pozycje w lokalnym podglądzie. Polecenie pokazania kontrolki przechodzi do niej w prawdziwym Excelu. **Uruchomienie przypisanego makra** jest osobną akcją dla obsługiwanych odwołań `OnAction`; nie odtwarza `Application.Caller` ani zdarzeń ActiveX. Przy takich zależnościach użyj rzeczywistego przycisku w Excelu. Nie są emulowane wszystkie rysunki, kontrolki ActiveX ani niestandardowe wstążki.

Opcja uruchomienia makr otwarcia pozwala wykonać inicjalizację skoroszytu. Nie łączy się jej z początkowym przeniesieniem lokalnych edycji, ponieważ `Workbook_Open` wykonałoby się przed nimi. Uruchamianie makr nie wymaga odczytu kodu VBA. Pivot nie zmienia ustawień Centrum zaufania; zachowuje oznaczenie pochodzenia kopiowanego pliku. Polecenia zmieniające skoroszyt i makra nie są automatycznie ponawiane po błędzie. Jedynie wywołanie odrzucone przez zajętego Excela — zanim cokolwiek wykonał, np. podczas edycji komórki albo przy otwartym oknie — jest ponawiane do 15 s, a potem zgłaszane jako „Excel jest zajęty”. Zwykły błąd makra lub odczytu nie blokuje kolejnego **Uruchom** w połączonej sesji. Rozłączenie wymaga odzyskania połączenia, a częściowo przeniesione zmiany wymagają sprawdzenia kopii przed dalszą pracą. Zdarzenia arkusza mogą zmienić inne komórki lub wywołać działania poza plikiem; częściowo zastosowana operacja pozostaje jawna i wymaga sprawdzenia w tej samej sesji.

Zakładka **Diagnostyka VBA** ma opcję **Odczytaj kod VBA przed uruchomieniem makra**, domyślnie wyłączoną. Odczyt następuje lokalnie przed pojedynczym wywołaniem makra. **Odczytaj kod VBA** odczytuje gotową sesję bez uruchamiania makra. Gdy Excel odmawia programistycznego dostępu do projektu, Pivot próbuje odczytać źródło z zapisanej kopii XLSM, bez zmiany ustawień Office. Taki wynik dotyczy pliku na dysku, nie późniejszych niezapisanych zmian kodu. Kod podglądu pozostaje w pamięci bieżącego okna; nie trafia do historii kontrolera ani zewnętrznej usługi. Odmowa lub blokada projektu nie zatrzymuje żądanego makra.

Po odpowiedzi na pytanie w Pivocie diagnostyka pokazuje wysłaną odpowiedź i statyczne fragmenty źródła pasujące do tekstu pytania. Można również obejrzeć procedurę wejściową, miejsca wyjścia, obsługę błędów, zapis dokumentu i odczytane moduły. Nowe pytanie Excela ponownie otwiera **Komunikaty**. To podgląd kodu odczytanego przed wywołaniem, **nie debugger ani potwierdzenie wykonania konkretnej gałęzi**. Złożony tekst pytania, zewnętrzne dodatki i brak dostępu do części projektu mogą uniemożliwić dopasowanie. Odczyt jest ograniczony do 100 modułów i 500 tys. znaków; niepełny wynik jest oznaczony.

Przycisk **Pierdol to** jest widoczny w aktywnej sesji; obok znajduje się powód, jeżeli nie można go jeszcze użyć. Pivot zapamiętuje odczytane pytanie także wtedy, gdy odpowiadasz bezpośrednio w Excelu — nie zgaduje wtedy wybranej odpowiedzi. Ponowienie tego samego makra zachowuje ostatnie pytanie, a zmiana makra lub sesji usuwa ten kontekst. Po zakończeniu działania makra i zamknięciu jego pytań przycisk przygotowuje migawkę bieżącego skoroszytu i szuka rozpoznawalnego bloku walidacji odpowiadającego tekstowi pytania. Nie wymaga włączenia diagnostyki przed makrem. Przy rozłączonym Excelu można jawnie wybrać analizę **ostatniej zapisanej kopii sesji**, bez połączenia z Excelem. Podgląd wskazuje czas zapisu i informuje, że niezapisane zmiany nie są uwzględnione.

W zakładce **Komunikaty** dziennik pokazuje przebieg poleceń, pytania, potwierdzenie przekazania odpowiedzi do okna Excela, odzyskiwanie połączenia oraz wynik szukania PDF. Przy odmowie przygotowania zmiany widać etap analizy, moduł, procedurę, fizyczne numery wierszy i krótki fragment nierozpoznanego kodu, także przy instrukcjach rozpisanych na kilka linii. Liczniki informują, ile modułów, procedur i warunków sprawdzono oraz ile dopasowano do pytania. Fragment jest oznaczony jako **statyczny kod**, nie dowód wykonania wskazanej instrukcji. Dziennik i szczegóły można skopiować z okna; fragmenty VBA pozostają w jego pamięci i nie są automatycznie zapisywane w historii sesji ani wysyłane poza program.

Podgląd pokazuje dokładną zmianę VBA. **Zastosuj w nowej kopii** tworzy osobny XLSM bez uruchamiania makra. **Zastosuj i uruchom generator** pozwala wybrać miejsce zapisu PDF, tworzy zmienioną kopię, otwiera jej osobną sesję i zleca jedno uruchomienie tej samej procedury. Oryginał i dotychczasowa sesja pozostają zachowane. Kopia odpowiada pokazanej migawce, bez późniejszych edycji w Excelu. Utworzenie kopii ani zakończenie makra nie potwierdza wygenerowania PDF — potrzebny jest rzeczywisty nowy lub zmieniony plik.

Utrata połączenia z obiektem Excela jest pokazywana jako rozłączenie sesji. **Połącz ponownie z Excelem** szuka dokładnie tej kopii sesji w tej samej instancji; w razie utraty obiektu COM, również błędu RPC `0x800706E6`, może odtworzyć połączenie przez okno własnego, nadal działającego procesu Excela. Odzyskiwanie obejmuje też błąd podczas odczytu arkuszy i kontrolek. Próba przez okno następuje najwyżej raz na polecenie; szczegóły błędu wskazują etap, na którym się zatrzymała. Nie otwiera pliku ponownie ani nie powtarza makra. Jeżeli przed poleceniem wykryto, że Excel przeładował skoroszyt, Pivot odświeża powiązanie i kontrolki, ale zatrzymuje zlecone polecenie. Sprawdź wynik poprzedniej próby przed kolejnym uruchomieniem.

Analizator rozpoznaje ograniczony zestaw bloków warunkowych, a nie dowolny program VBA. Zachowuje warunek walidacji `If` i jego wywołania, w tym odczyty arkusza oraz przygotowanie danych. Wyłącza rozpoznany komunikat i wyjście z procedury; dodatkowe operacje w pomijanym fragmencie zatrzymują propozycję. Obsługuje również `VBA.MsgBox`, bezpośrednie `If MsgBox(...)`, instrukcje kontynuowane w kolejnych wierszach oraz składanie tekstu w lokalnej zmiennej String. Gdy odpowiedź jest zapisywana do lokalnej zmiennej liczbowej lub VbMsgBoxResult, podgląd pokazuje przypisanie wartości kontynuacji 6/7 wynikającej z rozpoznanego warunku wyjścia. To celowa zmiana w kopii, a nie odczyt odpowiedzi użytkownika. Przygotowanie tekstu pozostaje wykonywane. Rozpoznaje też komunikaty składane z `Chr`/`ChrW` dla znaków nowego wiersza i tabulacji oraz standardowych konwersji. Przy kilku dopasowaniach wymaga wskazania konkretnej propozycji; przy nieobsługiwanym kodzie, ochronie projektu lub podpisie nie wykonuje zmiany. Odmowa analizy wskazuje procedurę i wiersz, jeśli te dane są dostępne; powtarzające się przyczyny są grupowane. Modyfikacja odbywa się lokalnie w pliku; Pivot nie włącza programistycznego dostępu w Centrum zaufania. Kopia zachowuje pozostałe części XLSM i znane pliki towarzyszące. Źródło VBA jest zapisywane bez starego kodu skompilowanego, aby Excel mógł je ponownie skompilować. Osobny przycisk **Otwórz zmienioną kopię w Pivocie** otwiera kopię bez uruchamiania generatora. Pominięcie walidacji nie uzupełnia brakujących danych i nie potwierdza poprawności ani utworzenia PDF. Polityki uruchamiania makr nadal obowiązują.

Analiza rozpoczyna się od konkretnego wywołania `MsgBox` i najmniejszego powiązanego warunku. Parser zachowuje fizyczne wiersze źródła i rozróżnia lokalne nazwy. Dla rozpoznanych bezpośrednich wywołań podgląd pokazuje możliwą ścieżkę od makra; jest to analiza źródła, nie ślad wykonania. Przy prostych walidatorach Boolean potrafi pominąć wkład jednej kontroli do wyniku, zachowując wcześniejsze `False` z innych kontroli. Nie dodaje bezwarunkowego `True`. Rozpoznane aktywowanie arkusza i przewijanie pozostają w pierwotnej kolejności; operacje zapisu i nierozpoznane skutki nie są usuwane. Krótkie odrzucone bloki są pokazywane w całości.

Kopia źródła i przygotowany plan są niezmienne i powiązane sumą SHA-256. Koordynator rozdziela stan połączenia, wynik wywołania makra i wynik dokumentu. Każda próba uruchomienia otrzymuje własny identyfikator; spóźnione odpowiedzi wcześniejszej próby są odrzucane. Samo odzyskanie połączenia nie uruchamia makra ponownie. Brak odpowiedzi po utracie połączenia oznacza nieznany wynik wykonania.

Kopiowane szczegóły analizy obejmują również przygotowane propozycje: różnicę kodu, zachowane i pomijane wiersze, wpływ na wynik walidacji oraz możliwe i nieustalone wywołania. Pozwala to porównać analizę z późniejszą próbą bez przedstawiania statycznego źródła jako wykonywanego kodu.

Pliki towarzyszące, np. szablon Worda, można wskazać przy otwieraniu sesji. Wybrane pliki są kopiowane obok skoroszytu z zachowaniem nazw; Pivot nie odgaduje wszystkich zależności VBA ani nie zmienia ścieżek zapisanych w makrze. Makro może uruchomić zainstalowanego Worda, ale jego okna i zapisy obsługuje sam Office. Pivot nie przejmuje ani nie zamyka cudzych procesów Worda.

Standardowe okna komunikatów należące do tej sesji pojawiają się w ramce Pivot wraz z rzeczywistymi odpowiedziami. Przy pytaniu „Czy przerwać sprawdzanie?” odpowiedź **Nie** oznacza, że nie prosisz o przerwanie. Nie ma dodatkowego przycisku „Pomiń”. Aktywne pytanie ma pierwszeństwo przed wcześniejszym błędem, którego pełne szczegóły pozostają dostępne. To nie zmienia warunków w kodzie VBA i nie gwarantuje, że skrypt mimo błędu wygeneruje raport. Niestandardowe formularze obsługujesz w widocznym Excelu. Odpowiedź jest przekazywana wyłącznie po kliknięciu, po ponownym sprawdzeniu aktualnego okna i procesu.

Przed uruchomieniem przycisku z **PDF** w podpisie Pivot pyta o nazwę i miejsce zapisania dokumentu. Można też ustawić je w **Plik i opcje → Zapis PDF z makra**. Anulowanie wyboru zatrzymuje uruchomienie tego przycisku. Po powrocie z makra Pivot sprawdza nowe i zmienione pliki PDF bezpośrednio w folderze sesji. Gdy znajdzie jeden potwierdzony plik i skan jest pełny, kopiuje go do wskazanego miejsca, zachowując plik sesji; pokazuje ścieżkę i przycisk otwarcia. Wiele wyników wymaga wyboru. Błąd makra lub niepełny skan wymagają ręcznego wskazania wyniku.

Zakończenie makra bez błędu COM nie potwierdza wygenerowania PDF. Jeśli nie znaleziono pliku, Pivot pokazuje tę informację. Makro może zapisywać poza folderem sesji lub skończyć zapis później; przycisk **Wskaż PDF…** pozwala wybrać taki dokument i zapisać go pod własną nazwą. Dotyczy to też makr uruchomionych bezpośrednio w Excelu. Wybrane miejsce zapisu w Pivocie nie zmienia ścieżek zapisanych w VBA. Przed zastąpieniem pliku docelowego Pivot sprawdza strukturę PDF, odczyt stron i ich zawartości, liczbę stron oraz niezmienność kopiowanych bajtów. Używa biblioteki [pypdf](https://pypdf.readthedocs.io/en/stable/modules/PdfReader.html), przygotowywanej wraz z obsługą Excela z wybranego źródła pakietów. Brak biblioteki lub nieudany odczyt nie daje potwierdzenia PDF. Limit kopiowania wynosi 256 MiB; sprawdzenie formatu nie ocenia poprawności treści raportu.

**PDF bieżącego arkusza** korzysta z drukowania Excela, niezależnie od przycisku generatora w skoroszycie. Zapisuje aktualną zawartość i obszar wydruku — nie potwierdza wykonania obliczeń generatora. **Zapisz kopię skoroszytu** zachowuje format, w tym VBA w XLSM. Można również zachować stan sesji w jej katalogu. Operacje potwierdzają sukces po sprawdzeniu pliku wynikowego.

Domyślnie katalogi sesji są w `%LOCALAPPDATA%\PivotStudio\excel-sessions` (lub pod własnym `--data-dir`). W **Plik i opcje** można wybrać inny folder kopii, np. lokalizację dopuszczoną w firmie. To lokalne ustawienie tego komputera; nie przechodzi w projekcie i nie zmienia zaufanych lokalizacji ani polityk Office. Katalogi pozostają po zamknięciu aplikacji, razem z dokumentami zapisanymi tam przez makra.

**Zapisz kopię sesji** zapisuje bieżący skoroszyt Excela pod tą samą ścieżką kopii. Później można jawnie wybrać zachowany plik i **wznowić tę kopię**. Pivot otwiera jej zapisany stan bez ponownego kopiowania oryginału i bez powtarzania wcześniejszych edycji. Sprawdza zgodność źródła i lokalnych zmian oraz nie pozwala dwóm swoim sesjom jednocześnie używać tej samej kopii. Kopia po niepełnym przeniesieniu zmian wymaga sprawdzenia; nie staje się poprawną sesją tylko przez ponowne otwarcie. Zachowanie katalogu nie oznacza zapisania niezapisanej pamięci Excela. Pliki utworzone przez makro w innych miejscach pozostają pod kontrolą tego makra.

Przy błędzie **„makro niedostępne”** Pivot pokazuje osobny krok sprawdzenia **tej samej kopii i sesji** w Excelu. Jeśli Excel udostępnia **Włącz zawartość**, użytkownik podejmuje tę decyzję w jego pasku zabezpieczeń. W przepływie **Zastosuj i uruchom generator** pojawia się następnie **Włączono zawartość — uruchom generator**: odświeża połączenie COM i zleca jedną próbę wykonania wybranej procedury, bez automatycznych powtórek. Pivot nie klika zgody i nie zmienia polityk Office. Obecność projektu VBA nie potwierdza zgody na wykonanie makra, a zwykły pasek zabezpieczeń Excela nie jest standardowym oknem pytania Tak/Nie. Działanie oryginału, zapisanej kopii oraz wywołania przez Pivot może się różnić; testy logiki aplikacji nie potwierdzają działania w firmowym środowisku Office i EDR.

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

Weryfikacja **0.7.4 na Windows, Python 3.12 i 3.14**: na każdej wersji ostatni pełny przebieg obejmował **618 testów rdzenia — 616 przeszło, 2 pominięto z powodu braku uprawnienia do dowiązań**. Regresje obejmują zapis i eksport, pivot z zakresu, liczby, daty, scalenia oraz katalog **49 295 obiektów**. Sprawdzono strumieniowy zapis katalogu przez protokół workera, ponowne otwarcie SQLite, ostatnią stronę, lokalne wyszukiwanie, szczegóły oraz JOIN. Zestawy cache obejmują 21 testów magazynu, 20 testów serwisu, 12 testów pobierania struktury i 11 testów mapy: anulowanie, wznowienie po restarcie, pierwszeństwo zadań użytkownika, ponowne użycie połączenia z nową transakcją dla każdej partii, zmiany bazy i generacji, limity pamięci i dysku, błędy częściowe, relacje spoza strony oraz odbudowę uszkodzonego cache. Zapytania Oracle wykonano na lokalnym słowniku testowym, bez serwera Oracle.

Nowe sprawdzenia obejmują 23 testy odczytu i eksportu pojedynczej komórki, 3 testy ich obsługi przez serwis, 6 testów danych XLSM oraz 2 testy układu mapy. Zweryfikowano tożsamość wiersza i kolumny, klucze złożone oraz binarne, anulowanie eksportu bez utraty poprzedniego pliku, ograniczenie podglądu, dokładne liczby JSON i dekodowanie Unicode na granicach porcji. Testy SQLite używają rzeczywistej bazy; zachowanie LOB-ów Oracle, Firebird i H2 sprawdzono na kontrolowanych odpowiednikach sterowników. Kontener XLSM zachowuje oryginalne bajty, a samo otwarcie danych nie wykonuje VBA.

Obsługę zewnętrznego Excela obejmują **28 testów kontrolera sesji, 12 testów przekazywania zmian, 9 testów zgodności importu Office i 11 testów komunikatów Win32**. Skrypt pomocnika przechodzi parser PowerShell, a filtr komunikatów COM kompiluje się i rejestruje w wątku STA. Sprawdzono osobny proces, trwałość wyników, zapis kopii, pliki towarzyszące, identyfikację skoroszytu/arkusza/kontrolki, konflikty komórek, częściowo wykonane edycje bez ponawiania, źródłowe SHA256 i pochodzenie wyników formuł. Standardowe okna Windows sprawdzono w rzeczywistym osobnym procesie. Funkcje skryptu Windows PowerShell wykonano na kontrolowanych obiektach testowych: odczyt zakresu i formatowania, literalny tekst, formuły, błędy komórek, ukrycie, scalenia, kontrolki oraz odrzucenie usuniętego arkusza zastąpionego nowym o tej samej nazwie. Transport poleceń testowano przez proces zastępczy. Na stanowisku testowym **nie ma zarejestrowanego Excel COM**; wykonania VBA ani eksportu przez prawdziwego Excela nie potwierdzono. Sprawdzono rzeczywisty start pomocnika i kontrolowane zakończenie przy braku COM.

Zestaw przygotowania obejmuje **70 wcześniej sprawdzonych testów** (pełny przebieg 69 oraz dodatkowa regresja i ponowny przebieg logiki instalatora po poprawce). Rzeczywista instalacja z publicznego PyPI w osobnym katalogu przygotowała wszystkie pięć profili, przeszła `pip check` i importy. Sprawdzono także rzeczywisty restart Qt z kopią niezapisanej pracy i potwierdzeniem startu nowego okna.

Ostatni pełny przebieg Qt w trybie `offscreen` objął **246 testów: 242 przeszły, 4 nie**. Te cztery niepowodzenia — skróty klawiaturowe, IME i układ nagłówka — należą do pięciu odtworzonych również na kodzie sprzed zmian Office (`a1e322a`); piąty, limit czasu wykazu technologii, tym razem nie wystąpił. We wcześniejszym przebiegu po podsumowaniu wystąpiła awaria procesu przy wyjściu (`0xC0000005`), z identyczną sygnaturą także sprzed zmian Office. W dwóch ostatnich przebiegach proces zakończył się zwykłym kodem nieudanych testów, ale przyczyna awarii wymaga osobnej diagnozy. Nie jest to w pełni zaliczony test całego GUI. Połączenie z firmowym Artifactory i rzeczywistymi instancjami H2, Firebird oraz Oracle wymaga dostępu do tych systemów.

Katalog Oracle, trwały cache i powiązane funkcje sprawdzono w **84 testach Qt — wszystkie przeszły**. Oprócz wcześniejszych 70 scenariuszy zestaw obejmuje 11 testów okna pełnej komórki, 2 testy otwierania XLSM i test rozmieszczania tabel po zmianie szerokości okna. Dwa testy przechodzą cały przepływ SQLite → worker → okno komórki → eksport pełnej wartości. Sprawdzono autoryzację odczytu w tle, pauzę, nieaktualne odpowiedzi, zachowanie kolumn sąsiada po odczycie szczegółów, błędy częściowe i zmianę generacji. „Uporządkuj” zmienia liczbę kart w rzędzie wraz z dostępną szerokością; wzrost kart po pobraniu kolumn nie powoduje kolizji z automatycznie rozmieszczonymi sąsiadami i zachowuje ręczne pozycje, zaznaczenie oraz powiększenie. Rzeczywisty widok Qt odczytał 49 295 nazw z pliku SQLite bez połączenia z Oracle. Interfejs nie modyfikuje odpowiedzi równolegle zapisywanej do cache. Scenariusze interfejsu Oracle korzystają z danych testowych.

Bieżącą obsługę Office sprawdzono dodatkowo w **45 testach Qt — wszystkie przeszły**. Obejmują kontrolki na arkuszu, źródłowe kolory i ukrycie, zapisane wyniki formuł, native snapshoty i jawne edycje, wybór arkusza, nieaktualne odpowiedzi, częściowe błędy, zapis przed zamknięciem i powrót do właściwego projektu. Okno sesji sprawdzono w obszarach roboczych od 683×364 do 1366×728, także na drugim monitorze i z czcionką powiększoną o 25%: stopka z **Uruchom** i **Zamknij** oraz odpowiedzi **Tak**/**Nie** pozostają w zasięgu. Testy obejmują również kolejkowanie poleceń podczas odczytu podglądu, nieblokujące błędy makra i odczytu, wybór spośród kilku przycisków i wiersz statusu; sprawdzono rzeczywiste renderowanie na syntetycznych danych. Kontroler Excel jest w tych testach zastępowany kontrolowanym odpowiednikiem, a zrzuty fixture nie stanowią dowodu wykonania VBA w Office.

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

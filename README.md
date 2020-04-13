### [Dokumentacja](https://docs.pytest.org/en/latest/contents.html)

Biblioteka służąca do uruchamiania testów. Jest niezwykle konfigurowalna. Pozwala na stworzenie własnego frameworka 
testowego.

#### Przydatne linki

- [dokumentacja pdf](https://buildmedia.readthedocs.org/media/pdf/pytest/latest/pytest.pdf)
- [pytest API](https://docs.pytest.org/en/latest/reference.html)
- [uruchamianie](https://docs.pytest.org/en/latest/usage.html)
- [dokumentowanie](https://docs.pytest.org/en/latest/doctest.html)
- [przykłady użycia](https://docs.pytest.org/en/latest/example/index.html)
- [tutoriale](https://docs.pytest.org/en/latest/talks.html)

### Budowanie struktury testów

Podział na [dwa przypadki]((https://pytest.readthedocs.io/en/reorganize-docs/new-docs/user/directory_structure.html)):

- testy mieszamy w foldery projektu
- umieszczamy je poza projektem w katalogu **tests** - 
[dobra praktyka]((https://docs.pytest.org/en/latest/goodpractices.html#tests-outside-application-code))
    - układ folderów i testów powinien odpowiadać temu z plikami projektu 
    - np. kiedy mamy moduł gui.py to odpowiednio tworzymy test_gui.py

#### Przykładowa struktura z kursu
```
vehicles
└── tests
    ├── truck
        ...
    └── sportcar
        ├── wheels
            └── test_durability.py
        ├── body
            ├── test_bumper.py
            └── test_doors.py
        └── engine
            └── test_engine.py
```

#### [Przykłady]((https://docs.pytest.org/en/latest/projects.html)) w istniejących projektach

- [circuits](https://github.com/circuits/circuits/tree/master/tests)
- [sentry](https://github.com/getsentry/sentry/tree/master/tests)
- [tox](https://github.com/tox-dev/tox/tree/master/tests)

### Uruchamianie

By uruchomić testy wystarczy wpisać `pytest` w katalogu gdzie one się znajdują, z tym że:

- nazwy plików ( test_\* ), klas ( Test\* ) oraz funkcji, metod ( test_\* ) powinny odpowiadać konwencji w nawiasach - 
można ją zmienić w pliku 
[pytest.ini](https://docs.pytest.org/en/latest/example/pythoncollection.html#changing-naming-conventions)
- nie zostały importowane moduły z pakietu - wyskoczy `ImportError`, by tego uniknąć możemy:
    - #### `python3 -m pytest tests`
        - wywołujemy w katalogu głównym projektu (gdzie tests to nazwa folderu z testami, zyskają 
    one dostęp do modułów projektu z katalogu `src`, np. o nazwie `memorizeIT`)
    - #### `pip install -e .` 
        - wywołujemy w katalogu głównym projektu jeśli mamy plik `setup.py`. Możemy wtedy zainstalować pakiet w wersji 
        `--extended`. Nie zostanie on umieszczony w systemie, ale kod z folderu projektu, nad którym pracujemy będzię 
        **zlinkowany**. Można go wtedy wywoływać tak jak pakiety zainstalowane normalnie. Przy czym wprowadzenie 
        jakichkolwiek zmian w projekcie, będzię miało miejsce natychmiast (bez przeinstalowania).
    - #### użycie tox
        - zalecane, *dobra praktyka*

#### Flagi

- `-h` : help
- `-v` : wyświetla detale
- `-s` : standard out, wyświetla print
- `--markers` : pokazuje dostępne markery
- `--fixtures` : pokazuje dostępne fixtury
- `-rs` : wyświetla opis z markerów skip podczas przebiegu testów
- `-rx` : wyświetla opis z markerów xfail podczas przebiegu testów

### Konfiguracja

W katalogu gdzie zaczynają się nasze testy, np. `tests` umieszczamy plik konfiguracyjny `pytest.ini` (lub dodajemy 
poniższe opcję do pliku `tox.ini`):
```ini
[pytest]
python_files = check_*
python_classes = *Check
python_functions = *_check

addopts = -v
testpaths = tests
markers = 
    smoke : All critical tests
    database : All database tests
    gui : All GUI tests
```

- `python_uruchomieniefiles`, `_classes`, `_functions` - pozwala na edycję domyślnych (w nawiasach) nazw plików (test_\*), klas 
(Test\*) oraz funkcji (test_\*) 
- `addopts` pozwala na dodawanie 
[predefiniowanych flag](https://docs.pytest.org/en/latest/customize.html#builtin-configuration-file-options) 
do wywołania przez użytkownika. Umieszczenie `-v` powyżej oznacza, że za każdym wywołaniem `pytest` w terminalu będzię 
wyświetlony wynik dla `pytest -v`
- `testpaths` określa nazwę katalogu z testami. Jeśli jej nie podamy wszystkie foldery zostaną przeszukane w celu 
odnalezienia testów
- `markers` - zawiera nazwy markerów użytych w testach oraz ich opis. Użytkownik może je wyświetlić wpisując: `pytest 
--markers`. Nie musimy zamieszczać opisu wszystkich, część może być tylko znana developerowi.

#### Użycie conftest.py i fixture

Każdy test znajdujący się w tym samym lub podrzędnym katalogu co `conftest.py` uzyskuje dostęp do funkcji zdefiniowanych 
w tym pliku (oznaczonych `@fixture`). Można je nadpisywać tworząc kolejny `conftest.py` w folderze wyżej i umieszczając 
w nim funkcję o tej samej nazwie. Więcej informacji w zakładce fixtures.

#### Dodawanie własnych argumentów

By stworzyć własną flagę używamy `parser.addoption`. Poniższy kod umieszczamy w `conftest.py`:
```python
def pytest_addoption(parser):
    parser.addoption('--env', action='store', default='dev', help='Sets environment for tests.')
```
Po wywołaniu `pytest -h` pojawi się w help - custom options nasza opcja wraz z opisem. Dostępne argumenty dla parsera 
można znaleźć w [dokumentacji](https://docs.python.org/3/library/argparse.html). Uwaga:

- słowo `parser` jest zarezerwowane w `pytest`, tak samo `request` służące do wczytania argumentów z linii komend. Tyczy 
się to również `config`. Nie należy przesłaniać *(shadow var)* żadnego z nich.
- dodawanie `default` może być błędem, nowy programista chcący uruchomić testy przykładowo w `qa` nie mając wiedzy, 
że domyślnie odpalą się w `dev` będzię błędnie interpretował wynik.

##### Przykładowe użycie

Tworzenie flag `--browser` i przekazywanie przeglądarek czy `--headless` nie otwierającej okien przeglądarki tylko 
przetwarzającej testy w tle (pamięci), `--env` określającej dane dla środowiska testowego, przykład poniżej:
```python    
class Config:
    def __init__(self, env):
        self.base_url = {'dev': 'https://mydev-env.com', 'qa': 'https://myqa-env.com'}[env]
        self.app_port = {'dev': 8080,'qa': 80}[env]
```
Dodajemy gdzieś klasę zawierającą naszą konfigurację, np. w `config.py`. Następnie tworzymy fixture pod definicją 
parsera (patrz powyżej `pytest_addoption`) z zasięgiem `session` - konfiguracja nie powinna się zmieniać w trakcie 
trwania sesji testowej. Zwraca ona nasz `Config` z odpowiednią wartością wczytaną z flagi `--env` w `conftest.py` :
```python
@fixture(scope='session')
def app_config(request):
    return Config(request.config.getoption('--env'))
```
Możemy używać naszej konfiguracji w testach, przykładowo:
```python
def test_environment_is_dev(app_config):
    assert app_config.base_url == 'https://mydev-env.com'
    assert app_config.app_port == 8080
```

##### [Więcej o argumentach](https://docs.pytest.org/en/latest/example/simple.html#control-skipping-of-tests-according-to-command-line-option)

#### [Dostępne opcje konfiguracji](https://docs.pytest.org/en/latest/reference.html#configuration-options)

### Raporty

Z przebiegu testów można generować raporty. Format wybieramy zależnie od potrzeb:

- #### html
    - czytelność dla odbiorcy
    - używamy dodatku `pytest-html` opisanego poniżej
    - generujemy raport: `pytest-html --html="results.html"`
- #### xml
    - intergracja z narzędziem CI, np. Jenkins
    - opis w kursie na udemy: *"Elegant Automation Frameworks with Python and Pytest"* (S02E05)
    - tworzymy raport: `pytest --junitxml="results.xml"`

### Dodatki

- #### [pytest-xdist](https://github.com/pytest-dev/pytest-xdist)
    - pozwala na wielowątkowe uruchomienie testów
    - instalacja: `pip install pytest-xdist`
    - użycie: `pytest -n4` gdzie po `n` podajemy ile procesów (lub jeśli nie ma tylu rdzeni to ile wątków) zostanie 
    uruchomione. Możemy wykorzystać `-nauto` do automatycznej konfiguracji.
- #### [pytest-socket](https://github.com/miketheman/pytest-socket)
    - dzięki niemu możemy łatwo zablokować połączenie internetowe
    - instalacja: `pip install pytest-socket`
    - użycie: `from pytest_socket import disable_socket`
- #### [pytest-html](https://github.com/pytest-dev/pytest-html)
    - umożliwia generowanie raportów `html` z testów
    - instalacja: `pip install pytest-html`
    - użycie: `pytest-html --html="results.html"`
- #### [pytest-cov](https://pytest-cov.readthedocs.io/en/latest/)
    - do sprawdzania pokrycia projektu testami
    - instalacja: `pip install pytest-cov`
    - użycie: `pytest --cov=project tests/`
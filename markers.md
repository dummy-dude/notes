### [Dokumentacja](https://docs.pytest.org/en/latest/mark.html) 

Markery umożliwiają dodanie tagów do testów. Dzięki temu możemy je grupować co pozwala uruchamiać wyłącznie poszczególne 
przypadki. Testy możemy również oznaczyć jako do pominięcia lub skazane na niepowodzenie. Poprzez parametryzację 
wykonujemy dany test wiele razy - dla każdego z podanych argumentów w markerze. Patrz przykłady poniżej.

Opis markerów zamieszczamy w pliku `ini`, np:
```
markers = 
    smoke : All critical tests
    database : All database tests
    gui : All GUI tests
```
Nie musimy opisywać wszystkich, część może być tylko znana developerowi. Do ich wyświetlenia używamy komendy: 
`pytest --markers`.

### Użycie

Przykładowy plik `test_me.py` oznaczony markerami:
```python
from pytest import mark

@mark.smoke
@mark.gui
def test_something():
    assert True
```
Do jednej funkcji, metody czy klasy możemy dodać dowolną ilość markerów. Jeśli klasa zostanie oznaczona to tyczy się to 
każdej dostępnej w niej metody, np. umieszczenie `@mark.skip` spowoduje, że żaden test z danej klasy nie zostanie 
wykonany.

### Uruchomienie

- `pytest -m nazwa` : uruchamia przypadki oznaczone markerem `@mark.nazwa`
- `pytest -m "smoke and gui"` : uruchamia przypadki posiadające oba markery (jak przykład powyżej)
- `pytest -m "smoke or gui"` : uruchamia przypadki oznaczone markerem `@mark.smoke` i te `@gui`
- `pytest -m "not gui"` : uruchamia wszystkie testy poza oznaczonymi markerem `@mark.gui`

#### Inne flagi

- `--markers` : wyświetla opis markerów zamieszczonych w pliku `ini`
- `-rs` : wyświetla opis z pominiętych markerów (skip)
- `-rx` : wyświetla opis z markerów skazanych na niepowodzenie (xfail)

### Predefiniowane markery

- `@mark.skip(reason='Fix in next sprint')` - służy do pomijania testów, informację dlaczego test został pominięty można 
umieścić w parametrze `reason` - opcjonalna (wyświetla się ją flagą `-rs`)
- `@mark.skipif(sys.platform=='win32', reason='Linux only tests')` - pomija test jeśli warunek jest spełniony, według 
kursu mało użyteczne - stosować `xfail`
- `@mark.xfail` - oznacza, że test się nie powiedzie, można jak przy `skipif` ustalić warunek (dla jakich przypadków 
zachodzi niepowodzenie) i podać przyczynę w `reason` (wyświetla ją flaga `-rx`)
- `@mark.parametrize('nazwa', ['arg1', 'arg2'])` - test oznaczony takim markerem uruchomi się tyle razy co liczba 
podanych argumentów w liście, przekazujemy je poprzez użytą powyżej nazwę: `def test_me(nazwa)`

### [Przykłady](https://docs.pytest.org/en/latest/skipping.html)
### [Parametryzacja](https://docs.pytest.org/en/latest/parametrize.html)
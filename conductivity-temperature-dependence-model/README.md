# Klasyczny model zależności przewodnictwa od temperatury

## 🚀 Opis

Projekt implementuje model Drudego - elektrony swobodnie przemieszczają się w przewodniku, będąc przyspieszane przez przyłożone pole elektryczne. Ich prędkość ulega jednak zmniejszeniu w wyniku zderzeń z drgającymi jonami w sieci krystalicznej.

Symulacja umożliwia wizualizację procesu, w którym nośniki poruszają się z katody do anody, a jednocześnie atomy wykonują chaotyczne ruchy wokół swoich początkowych pozycji. Cały proces jest monitorowany w czasie rzeczywistym, a dodatkowo możliwe jest śledzenie trajektorii nośników w postaci śladu.

### 🤖 Opis funkcjonalny

Atomy są rozmieszczone w regularnej siatce o wymiarach 10x5, gdzie każdy z nich jest przyczepiony do swojej początkowej pozycji za pomocą sprężyny (constraint). Ruch atomów jest symulowany poprzez chaotyczne, losowe wychylenia od początkowego położenia. Wychylenia te są generowane za pomocą losowego rozkładu Gaussa, który uwzględnia temperaturę systemu. Dzięki temu, każdy atom porusza się w sposób przypominający ruch cząsteczek, pod wpływem termicznych fluktuacji.

Nośniki ładunku (elektrony) pojawiają się w pobliżu katody, na losowej wysokości w obrębie okna symulacyjnego. Na nośniki działa siła elektryczna, która zależy od wartości pola elektrycznego oraz ładunku elementarnego elektronu. Nośniki poruszają się w kierunku anody, zyskując prędkość w wyniku działania pola elektrycznego. Po przebyciu całej szerokości planszy, uderzają w anodę, a następnie znikają, zostając zastąpione przez nowe nośniki.

### 🧑‍🔬 Użyte wzory

_Siła elektryczna działająca na nośnik:_
$ F = E \cdot q $

_Zasada ruchu nośników pod wpływem siły elektrycznej:_
$ F = m \cdot a $

_Ruch atomów - wychylenie gaussowskie:_
$ \Delta x = \sigma \cdot \mathcal{N}(0,1) $

_Zmiana prędkości nośnika w wyniku pola elektrycznego:_
$ v = v_0 + \frac{F \cdot t}{m} $

## 📝 Jak zbudować

- Sklonuj repozytorium
  Skopiuj kod źródłowy na swoje urządzenie, wykonując polecenie: \
  `git clone https://github.com/mbo009/conductivity-temperature-dependence-model`
- Przejdź do katalogu projektu
  Zmień bieżący katalog na główny folder projektu: \
  `cd conductivity-temperature-dependence-model`
- Zainstaluj potrzebne zależności
  `npm install`
- Uruchom aplikacje
  `npm run dev`

## 📖 Instrukcja użytkownika

1. Wejdź na stronę aplikacji, `http://localhost:3000/conductivity-temperature-dependence-model/` w przypadku manualnie zbudowanego projektu lub [Github pages](https://mbo009.github.io/conductivity-temperature-dependence-model/)
2. Dobierz parametry:
   - Temperatura
   - Koncentracja nośników
   - Masa atomu
   - Pole elektryczne
   - Rozmiar atomu
   - Rozmiar nośnika

## 🛠️ Stack technologiczny

Front-end: Vue
Silnik symulacji: Matter.js
Zarządzanie pakietami: npm

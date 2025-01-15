# Klasyczny model zależności przewodnictwa od temperatury

Filip Ryniewicz, Miłosz Cieśla

## 🚀 Opis

Projekt implementuje model Drudego - elektrony swobodnie przemieszczają się w przewodniku, będąc przyspieszane przez przyłożone pole elektryczne. Ich prędkość ulega jednak zmniejszeniu w wyniku zderzeń z drgającymi jonami w sieci krystalicznej.

Symulacja umożliwia wizualizację procesu, w którym nośniki poruszają się z katody do anody, a jednocześnie atomy wykonują chaotyczne ruchy wokół swoich początkowych pozycji. Cały proces jest monitorowany w czasie rzeczywistym, a dodatkowo możliwe jest śledzenie trajektorii nośnika w postaci śladu. Pod oknem symulacji wyświetlana jest jego prędkość dryfu.

### 🤖 Opis funkcjonalny

Atomy są rozmieszczone w regularnej siatce o wymiarach 10x5, gdzie każdy z nich jest przyczepiony do swojej początkowej pozycji za pomocą sprężyny (constraint). Ruch atomów jest symulowany poprzez chaotyczne, losowe wychylenia od początkowego położenia. Wychylenia te są generowane za pomocą losowego rozkładu Gaussa, który uwzględnia temperaturę systemu. Wartość wychyleń jest skalowana przez współczynnik temperatury (`temperatureFactor`), który wpływa na amplitudę tych fluktuacji. Dzięki temu, każdy atom porusza się w sposób przypominający ruch cząsteczek, pod wpływem termicznych fluktuacji, a intensywność tych ruchów jest bezpośrednio związana z temperaturą systemu.

Nośniki ładunku (elektrony) pojawiają się w pobliżu katody, na losowej wysokości w obrębie okna symulacyjnego. Na nośniki działa siła elektryczna, która zależy od wartości pola elektrycznego oraz ładunku elementarnego elektronu. Nośniki poruszają się w kierunku anody, zyskując prędkość w wyniku działania pola elektrycznego. Po przebyciu całej szerokości planszy, uderzają w anodę, a następnie znikają, zostając zastąpione przez nowe nośniki.

Masy atomów oraz elektronów są wyrażone w jednostkach masy atomowej (u), a rzeczywiste proporcje między masami atomów a elektronów zostały zachowane, co zapewnia realizm w symulacji.

Aby umożliwić widoczny ruch nośników, wyliczona wartość siły działającej na nośniki została przeskalowana. Dzięki temu, pomimo tego, że siła zależna od pola elektrycznego i ładunku elementarnego elektronu jest zbyt mała, aby w naturalny sposób spowodować ich ruch w symulacji, przeskalowanie siły pozwala na uzyskanie odpowiedniej dynamiki nośników w modelu.

### 🧑‍🔬 Użyte wzory

_Siła elektryczna działająca na nośnik:_
$ F = E \cdot q $

_Zasada ruchu nośników pod wpływem siły elektrycznej:_
$ F = m \cdot a $

_Ruch atomów - wychylenie gaussowskie:_
$ \Delta x = \sigma \cdot \mathcal{N}(0,1) $

_Zmiana prędkości nośnika w wyniku pola elektrycznego:_
$ v = v_0 + \frac{F \cdot t}{m} $

Atomy przyciągane do pozycji początkowej (analogicznie do prawa Hooke'a):
$ F = -k \cdot x $

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

1. Wejdź na stronę aplikacji, `http://localhost:3000/conductivity-temperature-dependence-model/` w przypadku manualnie zbudowanego projektu lub na automatycznie wygenerowane [Github pages](https://mbo009.github.io/conductivity-temperature-dependence-model/)
2. Dobierz parametry:
   - Temperatura - Wartości wyrażone w ℃, minimalna wartość to absolutne zero natomiast maksymalna to 2000 ℃. Parametr ten pozwala na zwiększenie termicznych fluktuacji jonów.
   - Koncentracja nośników - Bazowo w modelu znajduje się 20 elektronów, zmiana tej wartości pozwala na przemnożenie ich ilości.
   - Masa atomu - Wartość wyrażona w jednostkach masy atomowej (u), przyjmuje minimalną wartość dla litu, który jest najlżejszym metalem przewodzącym, a maksymalną wartość dla osmium, będącego najcięższym metalem przewodzącym
   - Pole elektryczne - Wartość określająca intensywność pola elektrycznego w symulacji. Wartość ta wpływa na ruch nośników ładunku (np. elektronów) w kierunku elektrody, w zależności od jego natężenia.
   - Rozmiar atomu - Wartość zwiększająca promień atomu
   - Rozmiar nośnika - Wartość zwiększająca promień nośnika
3. Obserwuj symulację, w razie własnych potrzeb możesz włączać i wyłączać pole elektryczne lub przeciągać elementy modelu jeśli włączyłeś tryb interaktywny.

## 🛠️ Stack technologiczny

Front-end: Vue \
Silnik symulacji: Matter.js \
Zarządzanie pakietami: npm

## 📚 Bibliografia

- https://if.pw.edu.pl/~malys/w14-Prad-el.pdf
- https://pl.wikipedia.org/wiki/Model_Drudego
- https://openstax.org/books/fizyka-dla-szk%C3%B3%C5%82-wy%C5%BCszych-tom-2/pages/9-2-model-przewodnictwa-w-metalach
- https://si-644-archipelagfizyki.vercel.app/

# Vytvoření projektu

{% hint style="info" %}
V tomto návodu bude ukázána jednodušší varianta vytvoření projektu, než jaká byla prezentována na sobotním tutorialu.
{% endhint %}

## Prerekvizity - Instalace/lokalizace Python

### Instalace Python

Tuto sekci přeskočte, pokud již máte python nainstalovaný.

Stáhněte ([https://www.python.org/downloads/](https://www.python.org/downloads/)) a nainstalujte Python.

{% hint style="warning" %}
Při instalaci Python hned na úvodní obrazovce instalátoru důrazně doporučujeme zaškrtnout volbu `Add Python 3.xx to PATH`. Tato volba umožní, že nebudete pro příkazy Python muset složitě vyhledávat cestu, kde je python nainstalovaný.
{% endhint %}

### Lokalizace python.exe

Nyní musíte na počítači nalézt umístění souboru python.exe. Možnosti:

1. Spusťte příkazový řádek a zadejte příkaz `python`. Pokud se se vypíše další řádek s informací začínající ccac  `Python 3.x.y (v3.x.y:abcdef....)`, máte Python správně uložený v cestě Path. Nemusíte dále nic řešit.
2. Pokud se vypíše chybová hláška (že soubor nelze nalézt), nebo se otevře okno "Windows Store" s nabídkou instalace Pythonu (nic neinstalujte a zavřete jej), musíte zjistit lokaci `python.exe` ručně:
   1. Podívejte se, kam jste Python instalovali - typicky bude na cestě `C:\Python\...\Scripts\python.exe`  nebo `C:\Users<User>\AppData\Local\Programs\Python..\Scripts\python.exe` a. Uložte/zapište si celou nalezenou cestu včetně `...python.exe`.
   2. Nebo nalezněte `python.exe` přes vyhledávání souborů ve Windows. Opět si uložte/zapište celou nalezenou cestu včetně `...python.exe`.
   3. Nebo (možná nejjednodušší varianta) - odinstalujte a nainstalujte Python znovu, tentokráte se zaškrtnutou variantou "Add Python to PATH".

## Vytvoření projektu

### Vytvoření složky projektu

Vyberte/vytvořte složku, ve kterém budou uloženy soubory vašeho projektu - například `C:\Users\<user>\Documents\Repository\MyStreamlitProject`.

### Vytvoření prostředí pro běh projektu

{% hint style="info" %}
Na počítači můžete mít základní instalaci prostředí Python. Python funguje na principu balíčků, které dodáváte do prostředí. Pokud nechcete mít jedno velké prostředí se spoustou balíčků, lze od základního prostředí udělat tzv. virtuální prostředí, které slouží typicky pouze pro běh konkrétní aplikace a obsahuje balíčky pro tuto aplikaci. My v našem řešení budeme používat virtuální postředí přes tzv. `venv`.

V projektech je typické mít vlastní virtuální prostředí umístěné ve složce `venv` (odlišujte `venv` jako nástroj pro tvorbu prostředí a `venv` jako název složky).

Blíže k venv například viz [https://docs.python.org/3/library/venv.html](https://docs.python.org/3/library/venv.html).
{% endhint %}

Ve složce projektu otevřete příkazový řádek. Zadejte příkaz:

```powershell
python -m venv venv
```

Tím se po chvilce vytvoří složka s běhovým prostředím pro projekt (vizálně nebude vidět žádný výstup). Příkaz znamená:

* `python -m`  ... spusť v prostředí python...
* `venv` ... a v něm nástroj venv a ...
* `venv` ... nech ho udělat nové virtuální prostředí nazvané `venv`.

Zkontrolujte, že se ve vaší složce vytvořila nová složka `venv` s dalším obsahem. Následně aktivujeme environment dalším příkazem příkazové řádky:

```powershell
venv\Scripts\activate.bat
```

Zadání řádku se změní na:

```powershell
(venv) C:\Users\<user>\Documents\Repository\MyStreamlitProject>
```

Nyní nainstalujeme potřebné knihovny pomocí příkazu `pip install`. Zadáme instalaci projektů _streamlit_ a _opencv-python_ (knihovna pro práci s obrázky) příkazy (instalujeme knihovny potřebné v rámci našich tutorialů, instalovat můžete samozřejmě jakékoliv jiné, je-li třeba):

```powershell
pip install numpy
pip install opencv-python
pip install streamlit
pip list
```

{% hint style="info" %}
`pip` je balíčkovací systém Pythonu určený pro instalaci závislostí. Blíže například viz [https://packaging.python.org/en/latest/tutorials/installing-packages/](https://packaging.python.org/en/latest/tutorials/installing-packages/).
{% endhint %}

### Vytvoření souboru projektu

Do naší složky projektu vytvoříme soubor `main.py` (souborů projektů můžeme mít více a pak při spouštění pouze odkazujeme na správný soubor. Do souboru `main.py` vložíme triviální příklad na vyzkoušení chování:

```python
import streamlit as st

st.text("ahoj")
```

Následně zadáme do příkazové řádky spuštění programu:

```powershell
streamlit run main.py
```

Mělo by se otevřít okno prohlížeče s nápisem "Ahoj".

### Vytvoření dávkového souboru na jednoduché spuštění

Dávkový soubor, tzv. `bat` je soubor, který lze spustit (poklepáním myší) a vykonat operace v něm uvedené. Abychom při spouštění projektového souboru nemuseli pokaždé spouštět příkazový řádek a zapisovat do něj příkazy, uděláme si takový `bat` soubor, který nám prostředí spustí automaticky.

Začínáme ve složce projektu. Pro vytvoření BAT souboru potřebujeme:

* znát cestu k souboru aktivujícímu environment - viz výše, chceme soubor `venv\scripts\activate.bat`
* znát název spouštěného souboru - viz výše, chceme soubor `main.py`.

Do složky projektu vytvoříme soubor `start.bat` a do něj vložíme kód:

```
.\venv\scripts\activate.bat & streamlit run main.py
```

První část aktivuje environment z příkazového řádku, následuje znak `&` a druhá část spouští streamlit aplikaci přes projektový soubor `main.py`.

Nyní můžeme zkusit `bat` soubor spustit. Mělo by se opět otevřít okno prohlížeče a v něm dříve vytvořená aplikace.

{% hint style="info" %}
Streamlit spouští aplikaci jako webovou stránku a server poslouchající na konkrétním číselném portu. Streamlit nemůže být puštěn 2x na stejném portu. Pokud jste nezavřeli příkazový řádek, na kterém jsme ukazovali spuštění projektu dříve, nepůjde bat soubor spustit kvůli konfliktům portů.

Někdy se může stát, že jiná aplikace už používá port, který chce používat streamlit.  V tom případě můžete doplnit informaci o tom, na jakém portu chcete streamlit spustit upravením příkazu (číslo portu vyberte dle vlastního uvážení):

```
.\venv\scripts\activate.bat & streamlit run main.py --server.port=8765
```
{% endhint %}


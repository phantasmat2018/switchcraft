# Switchcraft

Switchcraft (*switch* + *witchcraft*) — перемикач розкладки клавіатури для Windows. Виправляє текст,
набраний не в тій розкладці (`ghbdtn` → `привет`, `руддщ` → `hello`), гарячими клавішами
(`Pause` — слово, `Ctrl+Pause` — рядок, `Shift+Pause` — виділене) або автоматично під час набору;
`Alt+Pause` скасовує останнє автоматичне виправлення. Мови: англійська, українська, російська — усі
встановлені розкладки, без додаткових налаштувань. У меню трею — пауза, скасування, **Convert to ▸ English /
Українська / Русский** для виділеного тексту і вимкнення автовиправлення для окремої програми.

## Встановлення

1. Завантажте **`SwitchcraftSetup.exe`** з [останнього релізу](https://github.com/phantasmat2018/switchcraft/releases)
   і запустіть. Потрібен лише .NET Framework 4.8 (він є в Windows 10/11), права адміністратора не потрібні.
   Windows може показати «Windows protected your PC» — програма не підписана сертифікатом:
   **More info → Run anyway**.
2. У вікні можна ввімкнути автозапуск, ярлик на Робочому столі і **локальну модель Laya** (визначення мови
   прямо на ПК, див. нижче). Програма ставиться в `%LOCALAPPDATA%\Programs\Switchcraft` і одразу запускається
   (іконка в треї).

Без вікон: `SwitchcraftSetup.exe --quiet [--no-autostart] [--desktop] [--laya]`.

**Оновлення:** меню трею → **Check for updates…** (програма звертається до GitHub лише тоді).
**Видалення:** Settings → Apps → Switchcraft → Uninstall — прибирає програму, ярлики, автозапуск, а також
(за замовчуванням) модель Laya і налаштування.

## Як визначається мова

- **Без AI** (з версії 1.2.0): вбудовані **відкриті словники** англійської, української й російської (~50–80 тис.
  слів кожна) і рушій, що оцінює слово як набране і в інших розкладках — після пробілу, Enter, Tab чи
  розділового знака слово, набране не в тій розкладці, виправляється, а розкладка перемикається. Працює з
  усіма трьома мовами одразу; слово, однакове для української й російської, іде в ту, якою ви друкували
  останньою. Нічого не потрібно встановлювати додатково.
- **AI** (Settings → Detect language with AI): TypeSafe Jev або Cloudflare Clef (хмара, потрібен ключ) чи
  **локальна модель Laya** (на вашому ПК, текст нікуди не надсилається).

### Звідки словники

Зібрані з лічильників слів відкритих джерел: [Tatoeba](https://tatoeba.org) (CC BY 2.0 FR),
[Mozilla Common Voice](https://github.com/common-voice/common-voice) (українські речення, CC0) і
[FrequencyWords](https://github.com/hermitdave/FrequencyWords) / OpenSubtitles 2018 (українські частоти,
CC BY-SA 4.0). Вбудований файл правил поширюється на умовах CC BY-SA 4.0 — подробиці в
[`dictionaries/NOTICE.txt`](dictionaries/NOTICE.txt).

## Локальна модель Laya

Ставиться інсталятором або пізніше: **Settings → Detect language with AI → Configure… → Provider: Local
model (Laya, on this PC) → Install**. Switchcraft сам:

1. завантажує модель із релізу [`laya-r3`](https://github.com/phantasmat2018/switchcraft/releases/tag/laya-r3)
   (≈600 МБ) і перевіряє її контрольну суму;
2. завантажує [uv](https://github.com/astral-sh/uv) і ставить через нього окремий Python 3.12;
3. ставить PyTorch 2.14.1 — з CUDA, якщо є відеокарта NVIDIA (≈2,5 ГБ), інакше версію для процесора
   (≈250 МБ) — і пакети з [`laya-requirements.txt`](laya/laya-requirements.txt) (Laya 0.4.1 з сервером);
4. прописує шляхи в налаштуваннях і перевіряє, що модель відповідає.

Усе лягає в `%LOCALAPPDATA%\Switchcraft\Laya` (близько 5 ГБ з CUDA, 1,5 ГБ без неї), нічого не змінюючи в
системі. Сервер (`laya-serve`) Switchcraft запускає сам, лише поки вибрано цю модель, і закриває разом із собою.

Вручну (те саме, що робить кнопка):

```powershell
$L = "$env:LOCALAPPDATA\Switchcraft\Laya"
uv venv --python 3.12 "$L\venv"
uv pip install --python "$L\venv\Scripts\python.exe" "torch==2.14.1+cu126" --index-url https://download.pytorch.org/whl/cu126
uv pip install --python "$L\venv\Scripts\python.exe" -r laya-requirements.txt
Expand-Archive switchcraft-laya-r3.zip "$L\model"
```

(без NVIDIA: `torch==2.14.1+cpu` з `https://download.pytorch.org/whl/cpu`), а в Configure… вказати
Server = `…\venv\Scripts\laya-serve.exe`, Model folder = `…\model`, Device = `cuda` або `cpu`.

### Звідки модель

Дотренований енкодер [jhu-clsp/mmBERT-base](https://huggingface.co/jhu-clsp/mmBERT-base) (MIT) з
кодом [Laya 0.4.1](https://huggingface.co/convaiinnovations/laya) (Apache-2.0) на реченнях
[Tatoeba](https://tatoeba.org) (CC BY 2.0 FR), перетворених на приклади набору не в тій розкладці.
Подробиці — у [`laya/NOTICE.txt`](laya/NOTICE.txt).

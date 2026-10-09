# Switchcraft

Switchcraft (*switch* + *witchcraft*) — перемикач розкладки клавіатури для Windows. Виправляє текст,
набраний не в тій розкладці (`ghbdtn` → `привет`, `руддщ` → `hello`), гарячими клавішами або
автоматично під час набору. Мови: англійська, українська, російська.

## Завантаження

Сторінка [Releases](https://github.com/phantasmat2018/switchcraft/releases):

- **`Switchcraft.exe`** — сама програма (один файл, потрібен лише .NET Framework 4.8, який є в Windows 10/11).
- **`switchcraft-laya-r3.zip`** — локальна модель Laya (≈600 МБ). Вручну її завантажувати не треба:
  програма робить це сама (див. нижче).

## Локальна модель Laya

Laya визначає, якою мовою ви пишете, прямо на вашому ПК — набраний текст нікуди не надсилається.

Встановлення на будь-якому ПК: **Settings → Detect language with AI → Configure… → Provider: Local
model (Laya, on this PC) → Install**. Switchcraft сам:

1. завантажує модель із релізу [`laya-r3`](https://github.com/phantasmat2018/switchcraft/releases/tag/laya-r3)
   і перевіряє її контрольну суму;
2. завантажує [uv](https://github.com/astral-sh/uv) і ставить через нього окремий Python 3.12;
3. ставить PyTorch 2.14.1 — з CUDA, якщо є відеокарта NVIDIA (≈2,5 ГБ), інакше версію для процесора
   (≈250 МБ) — і пакети з [`laya-requirements.txt`](laya/laya-requirements.txt) (Laya 0.4.1 з сервером);
4. прописує шляхи в налаштуваннях і перевіряє, що модель відповідає.

Усе лягає в `%LOCALAPPDATA%\Switchcraft\Laya` (близько 4 ГБ з CUDA, 1,5 ГБ без неї), нічого не
змінюючи в системі. Сервер (`laya-serve`) Switchcraft запускає сам, лише поки вибрано цю модель, і
закриває разом із собою.

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

## Character Dataset Preparation Workflow for ComfyUI / Workflow для подготовки датасета персонажа в ComfyUI

This workflow automatically prepares a dataset for training/fine-tuning models for a specific character: loads images, detects a person on them, crops and centers them, removes the background, generates detailed text descriptions, and saves `image + text` pairs to a folder in dataset format.  
Этот workflow автоматически подготавливает датасет для обучения/дообучения моделей под конкретного персонажа: загружает изображения, находит на них человека, вырезает и центрирует его, удаляет фон, генерирует подробные текстовые описания и сохраняет пары `картинка + текст` в папку в формате датасета.

### Main Workflow Stages / Основные этапы работы

- **Image Loading / Загрузка изображений**
  - `LoadImage` can load a single image. / Через `LoadImage` можно загружать отдельное изображение.
  - `LayerUtility: LoadImagesFromPath` can batch load images from a specified folder (e.g., frames or raw character references). / Через `LayerUtility: LoadImagesFromPath` можно массово загружать изображения из указанной папки (например, кадры или сырые референсы персонажа).
  - The `ComfySwitchNode` (`switch: is mass loading`) allows you to select the mode: / Узел `ComfySwitchNode` (`switch: is mass loading`) позволяет выбрать режим:
    - **false** — uses single `LoadImage`. / **false** — используется одиночный `LoadImage`.
    - **true** — uses images from folder (`LoadImagesFromPath`). / **true** — используются изображения из папки (`LoadImagesFromPath`).

- **Person Detection and Cropping / Поиск и кроп человека**
  - `ToolYoloCropper` detects a **person** object on the image and creates a **square crop** with a specified padding. / `ToolYoloCropper` ищет на изображении объект типа **person** и делает **квадратный кроп** с заданным отступом (padding).
  - The square crop is used as the basis for further processing. / На выходе используется квадратный кроп как основа для дальнейшей обработки.

- **Resizing / Изменение размера**
  - `Image Resize (rgthree)` scales the square crop (default to 1024×1024 with `pad` mode), bringing all images to a uniform size. / `Image Resize (rgthree)` масштабирует квадратный кроп (по умолчанию до 1024×1024, с режимом `pad`), приводя все изображения к единому размеру.

- **Background Removal / Удаление фона**
  - `InspyrenetRembg` removes the background by building a mask of the object (character). / `InspyrenetRembg` вырезает фон, строя маску объекта (персонажа).
  - `LayerUtility: ImageRemoveAlpha` removes the alpha channel and fills the background with a specified color (default white), producing a clean RGB image of the character. / `LayerUtility: ImageRemoveAlpha` убирает альфа-канал и заполняет фон заданным цветом (по умолчанию белым), получая чистое RGB-изображение персонажа.

- **Description Generation (Captioning) / Генерация описаний (captioning)**
  - `DownloadAndLoadFlorence2Model` loads and initializes the `Florence-2` model (default `microsoft/Florence-2-base`). / `DownloadAndLoadFlorence2Model` загружает и инициализирует модель `Florence-2` (по умолчанию `microsoft/Florence-2-base`).
  - `Florence2Run` generates a **detailed caption** for each processed image (`more_detailed_caption` mode). / `Florence2Run` генерирует **детализированный caption** для каждого обработанного изображения (режим `more_detailed_caption`).
  - The `PrimitiveString` node (`Character Name`) sets the **character name** (e.g., `mmmargosha`). / Узел `PrimitiveString` (`Character Name`) задаёт **имя персонажа** (например, `mmmargosha`).
  - `StringConcatenate` combines: / `StringConcatenate` объединяет:
    - character name; / имя персонажа;
    - generated caption from `Florence2Run`; / сгенерированный caption от `Florence2Run`;
    - delimiter `, ` is used between them. / между ними используется разделитель `, `.
  - The resulting string is the final text prompt for the image. / Полученная строка — это финальный текстовый промпт для изображения.
  - The `JC` (JoyCaption) node is present in the workflow and can be used for alternative or additional captioning, but in the current configuration the main text comes from `Florence2Run`. / Узел `JC` (JoyCaption) присутствует в workflow и может быть использован для альтернативного или дополнительного captioning, но в текущей конфигурации основной текст берётся из `Florence2Run`.

- **Dataset Saving / Сохранение датасета**
  - `SaveImageTextDataSetToFolder` saves: / `SaveImageTextDataSetToFolder` сохраняет:
    - processed images after background removal; / обработанные изображения после удаления фона;
    - corresponding text descriptions (prompts); / соответствующие им текстовые описания (prompts);
    - to the specified folder in `image + text` format suitable for training/fine-tuning. / в указанную папку в формате набора `image + text`, подходящего для тренировки/дообучения.
  - In the node you can set: / В узле можно задать:
    - **folder_name** — where to save the dataset; / **folder_name** — куда сохранять датасет;
    - **filename_prefix** — file name prefix (default matches character name, e.g., `mmm`). / **filename_prefix** — префикс имени файла (по умолчанию совпадает с именем персонажа, например `mmm`).

- **Preview / Превью**
  - `PreviewImage` shows the final character images. / `PreviewImage` показывает итоговые изображения персонажа.
  - `Prompt Preview` (`PreviewAny`) shows the final text prompts (combination of character name and caption). / `Prompt Preview` (`PreviewAny`) показывает итоговые текстовые промпты (объединение имени персонажа и caption-а).
  - A separate `PreviewImage` is used to visualize the mask (via `MaskToImage`). / Отдельный `PreviewImage` используется для визуализации маски (через `MaskToImage`).

### Required Extensions / Nodes / Требуемые расширения / узлы

For the workflow to work correctly in ComfyUI, the following packages/nodes must be installed:  
Для корректной работы workflow в ComfyUI должны быть установлены следующие пакеты/узлы:

- **ComfyUI Core Nodes / Базовые узлы ComfyUI**
  - `LoadImage`, `SaveImageTextDataSetToFolder`, `PreviewImage`, `PreviewAny`, `StringConcatenate`, `PrimitiveString`, `MaskToImage`, `ComfySwitchNode`.
- **ComfyUI-Yolo-Cropper**
  - `ToolYoloCropper` node for person detection and cropping. / Узел `ToolYoloCropper` для детекции и кропа человека.
- **rgthree nodes**
  - `Image Resize (rgthree)` for image scaling. / `Image Resize (rgthree)` для масштабирования изображений.
- **comfyui-inspyrenet-rembg**
  - `InspyrenetRembg` for background removal by person. / `InspyrenetRembg` для вырезания фона по человеку.
- **comfyui_layerstyle**
  - `LayerUtility: LoadImagesFromPath` for batch loading images from folder. / `LayerUtility: LoadImagesFromPath` для массовой загрузки изображений из папки.
  - `LayerUtility: ImageRemoveAlpha` for alpha channel removal and background filling. / `LayerUtility: ImageRemoveAlpha` для удаления альфа-канала и заливки фона.
- **comfyui-florence2**
  - `DownloadAndLoadFlorence2Model` and `Florence2Run` for generating detailed captions. / `DownloadAndLoadFlorence2Model` и `Florence2Run` для генерации подробных caption-ов.
- **comfyui-joycaption** (optional, `JC` node already used) / (опционально, уже использован узел `JC`)
  - For additional/alternative captioning (optional). / Для дополнительного/альтернативного captioning (по желанию).

### How to Use the Workflow / Как использовать workflow

- **1. Installing Extensions / Установка расширений**
  - Install the extensions listed above in the `custom_nodes` folder of your ComfyUI installation, following the instructions from the respective repositories. / Установите перечисленные выше расширения в папку `custom_nodes` вашей установки ComfyUI, следуя инструкциям соответствующих репозиториев.
  - Restart ComfyUI so the nodes appear in the interface. / Перезапустите ComfyUI, чтобы узлы появились в интерфейсе.

- **2. Loading the Workflow / Загрузка workflow**
  - Import the `dataset-workflow.json` file via the `Load` / `Open` menu in ComfyUI. / Импортируйте файл `dataset-workflow.json` через меню `Load` / `Open` в ComfyUI.
  - Make sure all nodes are marked as found (no `missing node` errors). / Убедитесь, что все узлы помечены как найденные (без ошибок `missing node`).

- **3. Configuring Input Data / Настройка входных данных**
  - In `LayerUtility: LoadImagesFromPath`: / В `LayerUtility: LoadImagesFromPath`:
    - specify the path to the folder with character images; / укажите путь к папке с изображениями персонажа;
    - if needed, configure the limit on the number of images to load and selection of every nth frame. / при необходимости настройте лимит количества загружаемых изображений и выбор каждого n-го кадра.
  - In `LoadImage` (if single mode is needed) select a test/reference image. / В `LoadImage` (если нужен одиночный режим) выберите тестовое/референсное изображение.
  - In `switch: is mass loading (ComfySwitchNode)`: / В `switch: is mass loading (ComfySwitchNode)`:
    - **true** — use batch loading from folder; / **true** — использовать пакет загрузки из папки;
    - **false** — use single `LoadImage`. / **false** — использовать одиночный `LoadImage`.

- **4. Character and Save Configuration / Настройка персонажа и сохранения**
  - In `Character Name (PrimitiveString)` set the **character name** (it will be added to each text prompt and file prefix). / В `Character Name (PrimitiveString)` задайте **имя персонажа** (оно попадёт в каждый текстовый промпт и в префикс файлов).
  - In `SaveImageTextDataSetToFolder`: / В `SaveImageTextDataSetToFolder`:
    - `folder_name` — target folder for saving the dataset (relative to ComfyUI's main output directory); / `folder_name` — целевая папка для сохранения датасета (относительно основного каталога вывода ComfyUI);
    - optionally configure `filename_prefix` (e.g., short character name). / при желании настройте `filename_prefix` (например, краткое имя персонажа).

- **5. Running / Запуск**
  - Start execution from any of the final nodes (`PreviewImage`, `SaveImageTextDataSetToFolder`, or `Prompt Preview`). / Запустите выполнение с любого из финальных узлов (`PreviewImage`, `SaveImageTextDataSetToFolder` или `Prompt Preview`).
  - Wait for all images to be processed. / Дождитесь завершения обработки всех изображений.

- **6. Result / Результат**
  - In the specified folder you will get a set of processed character images with normalized size, cleaned background, and associated detailed text descriptions. / В указанной папке вы получите набор обработанных изображений персонажа с нормализованным размером, очищенным фоном и связанными с ними подробными текстовыми описаниями.
  - This data can be used for training/fine-tuning models (LoRA, fine-tuning, etc.), as well as a clean character reference dataset. / Эти данные можно использовать для обучения/дообучения моделей (LoRA, fine-tuning и т.п.), а также как аккуратный референс‑датасет персонажа.


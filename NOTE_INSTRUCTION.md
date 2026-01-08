# Text for Note node in workflow / Текст для Note узла в workflow

Copy the following text and paste it into the **Note** node in ComfyUI:  
Скопируйте следующий текст и вставьте его в узел **Note** в ComfyUI:

---

## 📋 Usage Instructions / Инструкция по использованию workflow

### 🔄 Loading Mode Switch (Switch: is mass loading) / Переключатель режима загрузки (Switch: is mass loading)

**Important / Важно:** Before running, select the image loading mode:  
**Важно:** Перед запуском выберите режим загрузки изображений:

- **☑️ TRUE (enabled)** — Batch processing from folder / **☑️ TRUE (включено)** — массовая обработка из папки
  - Uses `LoadImagesFromPath` node / Используется узел `LoadImagesFromPath`
  - Specify the path to the image folder in `LayerUtility: LoadImagesFromPath` node / Укажите путь к папке с изображениями в узле `LayerUtility: LoadImagesFromPath`
  - All images from the specified folder will be processed / Обработаются все изображения из указанной папки

- **☐ FALSE (disabled)** — Single image processing / **☐ FALSE (выключено)** — обработка одного изображения
  - Uses `LoadImage` node / Используется узел `LoadImage`
  - Select one image via the upload button in `LoadImage` node / Выберите одно изображение через кнопку загрузки в узле `LoadImage`
  - Only the selected image will be processed / Обработается только выбранное изображение

### ⚙️ What to configure / Что нужно настроить:

1. **Switch: is mass loading** — select mode (TRUE/FALSE) / выберите режим (TRUE/FALSE)
2. **Character Name** — enter character name (will be added to each caption) / введите имя персонажа (будет добавлено к каждому caption)
3. **LoadImagesFromPath** — specify folder path (if using batch loading) / укажите путь к папке (если используете массовую загрузку)
4. **SaveImageTextDataSetToFolder** — specify folder for saving dataset / укажите папку для сохранения датасета

### 🚀 After configuration, run the workflow! / После настройки запустите workflow!

---

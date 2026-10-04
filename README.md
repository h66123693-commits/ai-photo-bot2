import os
from io import BytesIO

from PIL import Image, ImageEnhance, ImageFilter
from telegram import Update
from telegram.ext import Application, MessageHandler, ContextTypes, filters

TOKEN = os.getenv("BOT_TOKEN")


async def improve_photo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    photo = update.message.photo[-1]

    file = await photo.get_file()
    data = await file.download_as_bytearray()

    image = Image.open(BytesIO(data)).convert("RGB")

    # تحسين بسيط للصورة: تكبير + حدة + إضاءة
    width, height = image.size
    image = image.resize((width * 2, height * 2), Image.Resampling.LANCZOS)
    image = ImageEnhance.Sharpness(image).enhance(1.8)
    image = ImageEnhance.Contrast(image).enhance(1.1)
    image = image.filter(ImageFilter.UnsharpMask(radius=1, percent=120, threshold=3))

    output = BytesIO()
    output.name = "enhanced.jpg"
    image.save(output, format="JPEG", quality=95)
    output.seek(0)

    await update.message.reply_photo(
        photo=output,
        caption="✅ تم تحسين الصورة"
    )


def main():
    app = Application.builder().token(TOKEN).build()

    app.add_handler(MessageHandler(filters.PHOTO, improve_photo))

    print("Bot is running...")
    app.run_polling()


if __name__ == "__main__":
    main()

import os
from flask import Flask, request, abort

from linebot.v3 import WebhookHandler
from linebot.v3.exceptions import InvalidSignatureError
from linebot.v3.messaging import (
    Configuration,
    ApiClient,
    MessagingApi,
    ReplyMessageRequest,
    TextMessage
)
from linebot.v3.webhooks import MessageEvent, TextMessageContent

app = Flask(__name__)

TOKEN = os.environ.get("CHANNEL_ACCESS_TOKEN")
SECRET = os.environ.get("CHANNEL_SECRET")
OWNER = os.environ.get("OWNER_USER_ID")

configuration = Configuration(access_token=TOKEN)
handler = WebhookHandler(SECRET)

prefix = "/"
admins = set()


def is_owner(user_id):
    return user_id == OWNER


def is_admin(user_id):
    return user_id in admins


def has_access(user_id):
    return is_owner(user_id) or is_admin(user_id)


def menu():
    return f"""╭──「 LINE PROTECT BOT 」──╮

👑 OWNER
• {prefix}admin ID
• {prefix}deladmin ID
• {prefix}admins
• {prefix}setcmd PREFIX

🛡️ ADMIN
• {prefix}protect on
• {prefix}protect off
• {prefix}getpict ID
• {prefix}gencover TEKS
• {prefix}invite NAMA

ℹ️
• help

╰──────────────────────╯"""


def command(user_id, text):
    global prefix

    text = text.strip()

    if text.lower() == "help":
        if has_access(user_id):
            return menu()
        return None

    if not text.startswith(prefix):
        return None

    data = text[len(prefix):].strip().split()

    if not data:
        return None

    cmd = data[0].lower()
    args = data[1:]

    # MEMBER TIDAK PUNYA AKSES
    if not has_access(user_id):
        return None

    # OWNER ONLY
    if cmd == "admin":
        if not is_owner(user_id):
            return "❌ Hanya Owner yang bisa menambah Admin."

        if not args:
            return f"Format: {prefix}admin ID"

        admins.add(args[0])
        return f"✅ Admin ditambahkan:\n{args[0]}"

    if cmd == "deladmin":
        if not is_owner(user_id):
            return "❌ Hanya Owner yang bisa menghapus Admin."

        if not args:
            return f"Format: {prefix}deladmin ID"

        admins.discard(args[0])
        return f"✅ Admin dihapus:\n{args[0]}"

    if cmd == "admins":
        if not is_owner(user_id):
            return "❌ Hanya Owner yang bisa melihat daftar Admin."

        if not admins:
            return "🛡️ Belum ada Admin."

        return "🛡️ DAFTAR ADMIN\n\n" + "\n".join(admins)

    if cmd == "setcmd":
        if not is_owner(user_id):
            return "❌ Hanya Owner yang bisa mengubah command."

        if not args:
            return f"Format: {prefix}setcmd PREFIX"

        prefix = args[0]
        return f"✅ Prefix berubah menjadi: {prefix}"

    # ADMIN
    if cmd == "protect":
        if not args:
            return f"Format: {prefix}protect on/off"

        if args[0].lower() == "on":
            return "🛡️ Protect ON"

        if args[0].lower() == "off":
            return "🛡️ Protect OFF"

        return f"Format: {prefix}protect on/off"

    if cmd == "getpict":
        if not args:
            return f"Format: {prefix}getpict ID"

        return "🖼️ Fitur getpict akan kita aktifkan setelah bot dasar berhasil."

    if cmd == "gencover":
        if not args:
            return f"Format: {prefix}gencover TEKS"

        teks = " ".join(args)
        return f"🎨 Permintaan gen cover diterima:\n{teks}"

    if cmd == "invite":
        if not args:
            return f"Format: {prefix}invite NAMA"

        nama = " ".join(args)
        return (
            f"👤 Target invite: {nama}\n\n"
            "Fitur invite akan kita sambungkan setelah bot dasar aktif."
        )

    return f"❌ Command tidak dikenal.\nKetik {prefix}help"


@app.route("/", methods=["GET"])
def home():
    return "LINE Protect Bot aktif."


@app.route("/callback", methods=["POST"])
def callback():
    signature = request.headers.get("X-Line-Signature")

    if not signature:
        abort(400)

    body = request.get_data(as_text=True)

    try:
        handler.handle(body, signature)
    except InvalidSignatureError:
        abort(400)

    return "OK"


@handler.add(MessageEvent, message=TextMessageContent)
def handle_message(event):
    user_id = event.source.user_id
    text = event.message.text

    reply = command(user_id, text)

    if reply is None:
        return

    with ApiClient(configuration) as api_client:
        api = MessagingApi(api_client)

        api.reply_message(
            ReplyMessageRequest(
                reply_token=event.reply_token,
                messages=[TextMessage(text=reply)]
            )
        )


if __name__ == "__main__":
    port = int(os.environ.get("PORT", 10000))
    app.run(host="0.0.0.0", port=port)

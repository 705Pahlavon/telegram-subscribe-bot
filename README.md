# telegram-subscribe-bot
import requests
import time

# 🔐 Ваш токен бота и username канала (без @)
TOKEN = '7669433341:AAFa0MIXvWx2SbS_oC8eWY8aNFI8uombAVI'
CHANNEL_USERNAME = 'SeZxx club'  # пример: 'mychannel'
API_URL = f'https://api.telegram.org/bot{TOKEN}/'
CHANNEL_LINK = f'https://t.me/+XGxJdz3kyAkyZDYy'

def get_updates(offset=None):
    params = {'timeout': 100, 'offset': offset}
    response = requests.get(API_URL + 'getUpdates', params=params)
    return response.json()

def send_message(chat_id, text, reply_markup=None):
    params = {
        'chat_id': chat_id,
        'text': text,
        'parse_mode': 'HTML'
    }
    if reply_markup:
        params['reply_markup'] = reply_markup
    requests.post(API_URL + 'sendMessage', json=params)

def check_subscription(user_id):
    url = f'{API_URL}getChatMember?chat_id=@{CHANNEL_USERNAME}&user_id={user_id}'
    res = requests.get(url).json()
    if 'result' in res:
        status = res['result']['status']
        return status in ['member', 'administrator', 'creator']
    return False

def main():
    last_update_id = None
    while True:
        updates = get_updates(last_update_id)
        if "result" in updates:
            for update in updates["result"]:
                if "message" in update:
                    message = update["message"]
                    chat_id = message["chat"]["id"]
                    user_id = message["from"]["id"]
                    text = message.get("text", "")

                    if text == "/start":
                        btn = {
                            "keyboard": [[{"text": "✅ Проверить подписку"}]],
                            "resize_keyboard": True,
                            "one_time_keyboard": False
                        }
                        send_message(
                            chat_id,
                            f"👋 Привет!\nДля использования бота подпишитесь на наш канал:\n👉 <a href='{CHANNEL_LINK}'>Перейти в канал</a>",
                            reply_markup=btn
                        )

                    elif text == "✅ Проверить подписку":
                        if check_subscription(user_id):
                            send_message(chat_id, "🎉 Спасибо! Вы подписаны на канал. Теперь вы можете использовать бота.")
                        else:
                            send_message(chat_id, "❗ Вы ещё не подписались на канал. Пожалуйста, подпишитесь и попробуйте снова.")

                last_update_id = update["update_id"] + 1
        time.sleep(1)

if __name__ == '__main__':
    main()

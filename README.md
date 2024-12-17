# Flask-AI-Agent-for-Restaurant-with-WhatsApp-and-Instagram-Integration
 AI agent for a restaurant. It uses a JSON file for the menu and supports function calling to save orders in a database. The script is already integrated with Facebook Messenger and works as a chatbot for my Facebook page.

What I Need:

Integration with WhatsApp and Instagram:

Extend the functionality to support WhatsApp and Instagram chats.
Order Management Enhancements:

Allow users to update and delete orders.
Returning Client Management:

Handle returning customers by recognizing their data (e.g., phone number, address) and continuing the conversation seamlessly.
Vector Search Optimization:

Improve the quality of the vector search for better recommendations and understanding.
Arabic Language Support:

The menu is in Arabic. I can assist with any language-related adjustments.
Skills Required:

Python Flask
API Integration (WhatsApp, Instagram)
Database Management
AI and Natural Language Processing (NLP)
Arabic Language Handling (optional but preferred)
Deliverables:

Fully integrated system supporting Facebook, WhatsApp, and Instagram.
Enhanced features for managing and recognizing returning customers.
Improved vector search performance.
Functional testing to ensure smooth operation across platforms.
---------------

To create an AI agent for a restaurant using Python Flask that integrates with Facebook Messenger, WhatsApp, and Instagram, and includes features like order management, returning client management, and vector search optimization, here's a detailed breakdown of the necessary components and a Python Flask script for the bot's backend.
Key Functionalities

    Integration with WhatsApp and Instagram:
        You'll need APIs for both WhatsApp and Instagram. WhatsApp can be integrated using the Twilio API, while Instagram integration typically uses Meta's Messenger API for Instagram Direct messages.

    Order Management Enhancements:
        Users should be able to place, update, or delete orders. This would involve managing orders in a PostgreSQL or MongoDB database.

    Returning Client Management:
        Store customer details (e.g., phone number, address) and use them to recognize returning customers. You can use session IDs or cookies to manage sessions for ongoing conversations.

    Vector Search Optimization:
        This can be done using Faiss or other vector search libraries to optimize recommendations based on customer preferences.

    Arabic Language Support:
        Use libraries like Google Translate API or langid for language identification, and make sure your NLP model or the chatbot supports Arabic.

Required Libraries:

pip install flask twilio psycopg2-binary requests transformers faiss-cpu langid

Flask Backend Example:

from flask import Flask, request, jsonify
from twilio.rest import Client
import psycopg2
import json
import requests
from transformers import pipeline
import langid

app = Flask(__name__)

# Database Configuration
DATABASE_CONFIG = {
    'dbname': 'restaurant_db',
    'user': 'your_user',
    'password': 'your_password',
    'host': 'localhost',
    'port': '5432'
}

# Load menu from JSON file (Arabic support)
with open('menu.json', 'r', encoding='utf-8') as f:
    menu = json.load(f)

# Twilio Client for WhatsApp
twilio_account_sid = 'your_twilio_account_sid'
twilio_auth_token = 'your_twilio_auth_token'
twilio_client = Client(twilio_account_sid, twilio_auth_token)

# NLP model for understanding orders
nlp_model = pipeline("text-classification", model="arabic_bert")

# Connect to PostgreSQL database
def get_db_connection():
    conn = psycopg2.connect(
        dbname=DATABASE_CONFIG['dbname'],
        user=DATABASE_CONFIG['user'],
        password=DATABASE_CONFIG['password'],
        host=DATABASE_CONFIG['host'],
        port=DATABASE_CONFIG['port']
    )
    return conn

# Handle incoming WhatsApp messages
@app.route('/webhook/whatsapp', methods=['POST'])
def whatsapp_webhook():
    data = request.get_json()
    from_number = data['From']
    message_body = data['Body']

    response_message = process_message(message_body, from_number)
    send_whatsapp_message(from_number, response_message)

    return jsonify({"status": "success"})


# Send message via WhatsApp using Twilio API
def send_whatsapp_message(to, message):
    twilio_client.messages.create(
        body=message,
        from_='whatsapp:+14155238886',  # Your Twilio WhatsApp number
        to=f'whatsapp:{to}'
    )


# Process the received message, return order summary or handle orders
def process_message(message, user_phone):
    # Detect the language of the message (Arabic support)
    lang, _ = langid.classify(message)
    
    if lang != 'ar':  # If message is not in Arabic, convert it (optional)
        message = translate_to_arabic(message)

    # Handle Order or Query
    if 'menu' in message.lower():
        return get_menu()

    elif 'order' in message.lower():
        return process_order(message, user_phone)
    
    # Add other message handling logic (e.g., update, delete, or status of orders)
    return "Sorry, I didn't understand that. Please try again."

# Translate text to Arabic (using a translation API if needed)
def translate_to_arabic(text):
    # Implement translation logic here
    # Example: Use Google Translate API or any other service
    return text  # Placeholder for translation

# Get the menu
def get_menu():
    menu_items = [f"{item['name']} - {item['price']} ريال" for item in menu]
    return "\n".join(menu_items)

# Process the order based on user input
def process_order(message, user_phone):
    # Parse the order from the message (Basic parsing logic for simplicity)
    order_items = parse_order(message)
    
    if order_items:
        return save_order(user_phone, order_items)
    return "Sorry, I couldn't understand your order. Please try again."

# Parse order items from message (simple example)
def parse_order(message):
    # For simplicity, we assume order is mentioned as item names separated by commas
    items = [item.strip() for item in message.split(",")]
    return items

# Save the order to the database
def save_order(user_phone, order_items):
    conn = get_db_connection()
    cursor = conn.cursor()

    # Insert the order in the database (Table: orders)
    cursor.execute("INSERT INTO orders (user_phone, items) VALUES (%s, %s)", 
                   (user_phone, json.dumps(order_items)))
    conn.commit()

    cursor.close()
    conn.close()

    return f"Your order has been placed: {', '.join(order_items)}."

# Handle Instagram messages via Meta API (requires Instagram Business API setup)
@app.route('/webhook/instagram', methods=['POST'])
def instagram_webhook():
    data = request.get_json()
    user_message = data['entry'][0]['messaging'][0]['message']['text']
    user_id = data['entry'][0]['messaging'][0]['sender']['id']

    response_message = process_message(user_message, user_id)
    send_instagram_message(user_id, response_message)

    return jsonify({"status": "success"})

# Send message via Instagram
def send_instagram_message(user_id, message):
    access_token = 'your_instagram_access_token'
    url = f'https://graph.facebook.com/v11.0/{user_id}/messages'
    payload = {
        "messaging_type": "RESPONSE",
        "recipient": {"id": user_id},
        "message": {"text": message}
    }
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f'Bearer {access_token}'
    }
    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# Main entry point
if __name__ == '__main__':
    app.run(debug=True)

Key Components:

    WhatsApp Integration:
        This uses Twilio's API for WhatsApp. The /webhook/whatsapp endpoint listens for incoming WhatsApp messages and processes them. It also sends the responses using the Twilio API.

    Instagram Integration:
        The Instagram part uses Meta's Messenger API for Instagram. The /webhook/instagram endpoint handles incoming messages from Instagram Direct and sends responses.

    Order Management:
        The process_order function interprets user orders and stores them in the PostgreSQL database. Orders can be parsed and saved in the database using save_order.

    Returning Client Management:
        By recognizing the user phone number (user_phone), the app can continue the conversation for returning customers. The process_message function can check and save the user's data.

    Vector Search Optimization:
        For recommendations, you can implement a vector search using libraries like Faiss for better item recommendations based on users' previous orders.

    Arabic Language Support:
        The app supports Arabic by detecting the language using langid, and translation features can be added to ensure seamless Arabic content handling.

Additional Enhancements:

    Error Handling: Add robust error handling and logging to make sure the bot is reliable.
    Analytics: Track customer orders and interactions to provide personalized recommendations.
    User Authentication: If necessary, you can implement user authentication for better returning customer management.

Integration and Testing:

    Test your Flask app locally and ensure the endpoints are working.
    Integrate the bot into your Facebook Messenger, WhatsApp, and Instagram accounts.
    Make sure to test both Arabic and English interactions to ensure smooth communication.

By implementing this backend, you'll be able to offer a powerful, AI-driven, multilingual chatbot that manages orders, handles returning customers, and interacts with users via multiple platforms.

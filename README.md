from flask import Flask, request, jsonify
import requests
from bs4 import BeautifulSoup

app = Flask(__name)

# Function to fetch Google auto-suggestions
def get_google_suggestions(query):
    base_url = "https://www.google.com/complete/search"
    params = {
        "q": query,
        "cp": "0",
        "client": "psy-ab",
        "xssi": "t",
    }

    try:
        response = requests.get(base_url, params=params)
        response.raise_for_status()
        data = response.text[6:]  # Removing ')]}\',\n' from the beginning
        suggestions = [item[0] for item in eval(data)][1]
        return suggestions
    except Exception as e:
        print("Error:", str(e))
        return []

@app.route('/suggestions', methods=['GET'])
def suggestions():
    query = request.args.get('query')
    if not query:
        return jsonify({"error": "Query parameter 'query' is required"}), 400

    suggestions = get_google_suggestions(query)
    return jsonify({"suggestions": suggestions})

if __name__ == '__main__':
    app.run(debug=True)

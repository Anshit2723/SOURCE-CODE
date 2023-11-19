# SOURCE-CODE

Query Interface code:
from flask import Flask, request, render_template
from elasticsearch import Elasticsearch

app = Flask(__name__)
es = Elasticsearch()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/search', methods=['POST'])
def search_logs():
    query = request.form['query']
    filters = {
        "level": request.form['level'],
        "message": request.form['message'],
        "resourceId": request.form['resourceId'],
        "timestamp": request.form['timestamp'],
        "traceId": request.form['traceId'],
        "spanId": request.form['spanId'],
        "commit": request.form['commit'],
        "metadata.parentResourceId": request.form['parentResourceId']
    }
    results = search(query, filters)
    return render_template('results.html', results=results)

def search(query, filters):
    must_queries = [{"query_string": {"query": query}}]

    for field, value in filters.items():
        if value:
            must_queries.append({"term": {field: value}})

    body = {
        "query": {
            "bool": {
                "must": must_queries
            }
        }
    }
    response = es.search(index='logs', body=body)
    return response['hits']['hits']

if __name__ == '__main__':
    app.run(port=3001)









 Log Ingestor :


 from flask import Flask, request, jsonify
from elasticsearch import Elasticsearch

app = Flask(__name__)
es = Elasticsearch()

@app.route('/ingest', methods=['POST'])
def ingest_log():
    try:
        log_data = request.get_json()
        es.index(index='logs', body=log_data)
        return "Log ingested successfully!"
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/bulk_ingest', methods=['POST'])
def bulk_ingest_logs():
    try:
        logs_data = request.get_json()
        if not isinstance(logs_data, list):
            return jsonify({"error": "Invalid data format. Expected a list of logs."}), 400

        for log_data in logs_data:
            es.index(index='logs', body=log_data)

        return "Logs ingested successfully!"
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(port=3000)

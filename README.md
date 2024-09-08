import time
from flask import Flask, request, jsonify, render_template, redirect

app = Flask(__name__)

# Configuration
MAX_REQUESTS = 5  # Maximum requests allowed per IP
TIME_FRAME = 60  # Time frame in seconds for rate limiting per IP
REQUEST_LIMIT = 100000  # Global request limit
DUMMY_SITE_URL = "https://example-dummy-site.com"  # URL for redirection

# Dictionary to store IPs and their request counts
request_counts = {}

# Variable to keep track of the total number of requests
global_request_count = 0

@app.before_request
def block_multiple_requests():
    global global_request_count
    ip = request.remote_addr
    current_time = time.time()
    
    # Check if the global request limit is exceeded
    global_request_count += 1
    if global_request_count > REQUEST_LIMIT:
        return redirect(DUMMY_SITE_URL, code=302)

    # Initialize request count and timestamp if IP is not in the dictionary
    if ip not in request_counts:
        request_counts[ip] = {'count': 1, 'first_request_time': current_time}
    else:
        # Check if the time frame has passed
        time_diff = current_time - request_counts[ip]['first_request_time']
        
        if time_diff < TIME_FRAME:
            # Increment request count
            request_counts[ip]['count'] += 1
            
            # Block IP if request limit is exceeded
            if request_counts[ip]['count'] > MAX_REQUESTS:
                return jsonify({"error": "Too many requests. Try again later."}), 429
        else:
            # Reset count and time if time frame has passed
            request_counts[ip] = {'count': 1, 'first_request_time': current_time}

@app.route('/')
def index():
    return render_template("index.html")

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')

# README.md
 
import pandas as pd
import base64
import re
import matplotlib.pyplot as plt
from PIL import Image
from io import BytesIO

# Function to parse log entries
def parse_log_entry(entry):
    log_dict = {}
    key_values = entry.split(',')
    for kv in key_values:
        if '=' in kv:
            key, value = kv.split('=')
            # Convert to appropriate data type
            if value.isdigit():
                log_dict[key] = int(value)
            elif re.match(r'^\d+?\.\d+?$', value):
                log_dict[key] = float(value)
            elif value.lower() in ['true', 'false']:
                log_dict[key] = value.lower() == 'true'
            elif value.lower() == 'null':
                log_dict[key] = None
            else:
                log_dict[key] = value
    return log_dict

# Function to check if a string is Base64 encoded
def is_base64(sb):
    try:
        if isinstance(sb, str):
            sb_bytes = bytes(sb, 'ascii')
        elif isinstance(sb, bytes):
            sb_bytes = sb
        else:
            raise ValueError("Input must be a string or bytes.")
        return base64.b64encode(base64.b64decode(sb_bytes)) == sb_bytes
    except Exception:
        return False

# Function to decode Base64 images
def decode_base64_image(encoded_data, output_file):
    try:
        image_data = base64.b64decode(encoded_data)
        image = Image.open(BytesIO(image_data))
        image.save(output_file)
        print(f"Image saved as {output_file}")
    except Exception as e:
        print(f"Error decoding image: {e}")

# Function to visualize data
def visualize_data(df):
    plt.figure(figsize=(10, 6))
    for column in df.select_dtypes(include=[float, int]).columns:
        df[column].plot(kind='line', label=column)
    plt.title('Sensor Data Visualization')
    plt.xlabel('Index')
    plt.ylabel('Values')
    plt.legend()
    plt.show()

# Read log file
log_file = 'iot_logs.txt'
with open(log_file, 'r') as file:
    log_entries = file.readlines()

# Extract and structure data
data = []
for entry in log_entries:
    data.append(parse_log_entry(entry.strip()))

# Convert to DataFrame
df = pd.DataFrame(data)

# Example of handling Base64 encoded image in logs
base64_image = '...'  # Replace with actual Base64 string from log
decode_base64_image(base64_image, 'decoded_image.png')

# Example of visualizing data
visualize_data(df)

# Example of handling web server logs (Apache/Nginx)
def parse_web_server_log(entry):
    pattern = r'(\d+\.\d+\.\d+\.\d+) - - \[(.*?)\] "(.*?)" (\d+) (\d+)'
    match = re.match(pattern, entry)
    if match:
        return {
            'IP': match.group(1),
            'Time': match.group(2),
            'Request': match.group(3),
            'Status': int(match.group(4)),
            'Size': int(match.group(5))
        }
    return None

web_log_file = 'web_server_logs.txt'
with open(web_log_file, 'r') as file:
    web_log_entries = file.readlines()

web_data = [parse_web_server_log(entry) for entry in web_log_entries if parse_web_server_log(entry)]
web_df = pd.DataFrame(web_data)

# Display web server log analysis
print(web_df['Status'].value_counts())
print(web_df['IP'].value_counts())

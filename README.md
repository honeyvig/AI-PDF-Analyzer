# AI-PDF-Analyzer
To create a web tool where you can upload a PDF file, use a GPT model to analyze it based on a preset prompt, and return the result, we need to break down the project into a few core steps:

    PDF Upload: Set up a file upload feature on the web page to allow users to upload a PDF file.
    PDF Text Extraction: Extract text from the uploaded PDF file.
    Interacting with GPT Model: Send the extracted text to a GPT model, using a preset prompt to analyze the text.
    Displaying the Result: Show the result from the GPT model on the web page.

Technologies Used:

    Flask: For the backend web framework in Python.
    PyPDF2 or PDFplumber: For extracting text from PDF files.
    OpenAI's GPT Model (using OpenAI API): To analyze the extracted text.
    HTML & JavaScript: For the front-end to upload the file and display the result.

Steps to Build the Web Tool:
Step 1: Install Required Packages

You will need the following libraries:

pip install flask openai PyPDF2

Step 2: Python Code for Backend (Flask App)

Here’s a simple Python Flask app to handle the PDF upload, extract text, and send it to OpenAI GPT model.

from flask import Flask, render_template, request, jsonify
import openai
import PyPDF2
import os

# Initialize Flask app
app = Flask(__name__)

# Set your OpenAI API key (replace with your actual OpenAI API key)
openai.api_key = 'your-openai-api-key'

# Upload folder for storing the uploaded PDFs temporarily
UPLOAD_FOLDER = 'uploads/'
if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# Helper function to extract text from the PDF
def extract_text_from_pdf(pdf_path):
    with open(pdf_path, 'rb') as file:
        reader = PyPDF2.PdfFileReader(file)
        text = ""
        for page_num in range(reader.numPages):
            text += reader.getPage(page_num).extract_text()
    return text

# Define the function to interact with GPT model
def analyze_with_gpt(text, prompt):
    # Create the final prompt by appending the preset prompt with the extracted text
    full_prompt = f"{prompt}\n\n{text}"
    
    # Call OpenAI's API to analyze the text with GPT-3
    response = openai.Completion.create(
        engine="text-davinci-003",  # You can use other models like "gpt-3.5-turbo" too
        prompt=full_prompt,
        max_tokens=500,  # You can adjust this based on how long the output should be
        n=1,
        stop=None,
        temperature=0.7
    )
    
    # Extract and return the result from GPT
    return response.choices[0].text.strip()

# Route to render the home page with the upload form
@app.route('/')
def home():
    return render_template('index.html')

# Route to handle the PDF upload and analysis
@app.route('/upload', methods=['POST'])
def upload_pdf():
    if 'pdf_file' not in request.files:
        return jsonify({"error": "No file part"})
    
    file = request.files['pdf_file']
    
    if file.filename == '':
        return jsonify({"error": "No selected file"})
    
    # Save the uploaded PDF file
    file_path = os.path.join(app.config['UPLOAD_FOLDER'], file.filename)
    file.save(file_path)
    
    # Extract text from the uploaded PDF
    extracted_text = extract_text_from_pdf(file_path)
    
    # Define your preset prompt (you can customize this as needed)
    prompt = "Analyze the following PDF text and summarize the key points:"
    
    # Use GPT model to analyze the extracted text based on the prompt
    result = analyze_with_gpt(extracted_text, prompt)
    
    # Return the result as a JSON response
    return jsonify({"result": result})

# Run the app
if __name__ == '__main__':
    app.run(debug=True)

Explanation of Code:

    Flask Setup: The Flask app handles both the home page (/) and the file upload endpoint (/upload).
    File Upload Handling: The upload_pdf() function handles the file upload, saves the PDF to a temporary folder, and then processes the PDF file.
    Text Extraction: We use the PyPDF2 library to extract text from the PDF file.
    GPT Interaction: After extracting the text, we send it to OpenAI's GPT-3 model using openai.Completion.create(). The result is returned as a summary or analysis of the text, based on the prompt you provide.
    Response: Once GPT processes the PDF content, the app returns the result in JSON format.

Step 3: Front-End (HTML + JavaScript)

Create a simple HTML form to upload the PDF and display the result.

templates/index.html:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF Analyzer</title>
</head>
<body>
    <h1>Upload a PDF to Analyze</h1>
    
    <form id="upload-form" enctype="multipart/form-data">
        <input type="file" id="pdf-file" name="pdf_file" accept="application/pdf" required>
        <button type="submit">Upload PDF</button>
    </form>

    <h2>Analysis Result:</h2>
    <pre id="result"></pre>

    <script>
        // Handle the form submission and PDF upload
        document.getElementById('upload-form').addEventListener('submit', async function(e) {
            e.preventDefault();
            
            const formData = new FormData();
            formData.append("pdf_file", document.getElementById('pdf-file').files[0]);
            
            try {
                const response = await fetch('/upload', {
                    method: 'POST',
                    body: formData
                });
                
                const data = await response.json();
                
                if (data.result) {
                    document.getElementById('result').textContent = data.result;
                } else {
                    document.getElementById('result').textContent = "Error analyzing the PDF.";
                }
            } catch (error) {
                console.error(error);
                document.getElementById('result').textContent = "An error occurred.";
            }
        });
    </script>
</body>
</html>

Explanation of HTML:

    File Upload Form: The form allows the user to upload a PDF file.
    JavaScript: Handles the form submission, sends the file to the backend via a POST request to /upload, and then displays the result returned by the backend (from the GPT model).

Step 4: Run the Flask App

To start the Flask app, navigate to the directory containing your app and run:

python app.py

This will start the web server, and you can open your browser to http://localhost:5000 to access the web tool.
Step 5: Testing the Tool

    Upload a PDF file via the form.
    The backend will process the file, extract the text, and send it to the GPT model.
    You’ll get the analysis result displayed on the page.

Conclusion:

This web tool allows users to upload PDF files, analyze them using a GPT model with a preset prompt, and return results in a structured way. You can further customize the prompts and results to fit your needs and improve the accuracy of the analysis based on the type of PDFs you are dealing with (e.g., legal documents, research papers, etc.).

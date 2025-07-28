# Use official Python image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements and install
COPY requirements.txt .
RUN pip install Flask==2.0.3 Werkzeug==2.0.3

# Copy app code
COPY app.py .

# Expose port 5000
EXPOSE 5000

# Run the app
CMD ["python", "app.py"]

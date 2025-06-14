FROM python:3.11-slim
WORKDIR /app
COPY app.py /app
RUN pip install streamlit
CMD ["streamlit", "run", "app.py", "--server.port=8501", "--server.enableCORS=false", "--server.enableWebsocketCompression=true"]

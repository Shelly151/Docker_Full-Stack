# Full Stack App Using Docker

This project demonstrates how to deploy a full-stack application using **Docker**, consisting of:
- A **PostgreSQL** database container
- A **Streamlit** frontend container
- A **Docker network** for seamless communication

## Prerequisites
Ensure you have the following installed on your system:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Basic knowledge of Streamlit and PostgreSQL

---

## 1. Create a Docker Network
We will create a **bridge** network to connect the database and frontend containers.

```sh
docker network create my_network
```

---

## 2. Set Up the Database Container

Run the following command to set up a PostgreSQL container:

```sh
docker run -d \
  --name my_postgres \
  --network my_network \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=adminpassword \
  -e POSTGRES_DB=mydb \
  postgres
```
This creates a PostgreSQL container named **my_postgres** connected to **my_network**.

---

## 3. Create the Streamlit App Container
### Dockerfile for Streamlit App
Create a **Dockerfile** in your Streamlit project folder:

```dockerfile
# Use the official Python image
FROM python:3.9

# Set the working directory
WORKDIR /app

# Copy the app files
COPY . .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose Streamlit port
EXPOSE 8501

# Run Streamlit
CMD ["streamlit", "run", "stream.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

Ensure your **requirements.txt** includes:
```txt
streamlit
psycopg2
```

### Build and Run the Streamlit Container

```sh
docker build -t my_streamlit_app .
```

Run the container and connect it to **my_network**:

```sh
docker run -d \
  --name streamlit_app \
  --network my_network \
  -p 8501:8501 \
  my_streamlit_app
```

---

## 4. Connect the Streamlit App to PostgreSQL
Create **stream.py**:

```python
import streamlit as st
import psycopg2

# Database connection
conn = psycopg2.connect(
    dbname="mydb",
    user="admin",
    password="adminpassword",
    host="my_postgres",  # Use the container name as the hostname
    port="5432"
)
cur = conn.cursor()

# Example query
cur.execute("SELECT version();")
db_version = cur.fetchone()

st.title("Streamlit App with PostgreSQL")
st.write("Connected to database:", db_version)

cur.close()
conn.close()
```

Now, you can test the setup by visiting:
```
http://localhost:8501
```

---

## 5. Create a Custom Bridge Network (Optional)
If you want a custom bridge network, run:

```sh
docker network create --driver bridge my_custom_network
```

---

## 6. Insert Dummy Data into PostgreSQL
### Access PostgreSQL Container
Run the following command to access the PostgreSQL container:

```sh
docker exec -it my_postgres psql -U admin -d mydb
```

### Create a Sample Table and Insert Data
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

INSERT INTO users (name, email) VALUES
('Alice Johnson', 'alice@example.com'),
('Bob Smith', 'bob@example.com'),
('Charlie Brown', 'charlie@example.com');

SELECT * FROM users;
```

---

## 7. Deployment
To deploy and run the application:
```sh
docker run --name streamlit_app --network my_network -p 8501:8501 my_streamlit_app
```

---

## Summary
1. **Create a Docker network**
2. **Deploy PostgreSQL in a container**
3. **Build and run the Streamlit container**
4. **Connect Streamlit to PostgreSQL**
5. **Insert and fetch data from PostgreSQL**
6. **Access the application at** `http://localhost:8501`

**Thank You! 🚀**


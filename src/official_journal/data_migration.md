# Migrating from JSON to PostgreSQL

1. Run the Docker Database
    ```bash
    docker compose exec -it db psql
    ```
2. Create the Extension and the Database
    ```SQL
    CREATE EXTENSION IF NOT EXISTS ai CASCADE;
    CREATE DATABASE kanoun;
    ```
3. Connect to the database
    ```psql
    \c kanoun
    ```
4. Create the `journals` table
    ```SQL
    CREATE TABLE journals (
        year INT,
        number INT,
        page INT,
        content TEXT
    );
    ```
5. Move the CSV database to Docker
    ```bash
    docker cp path/to/db.csv <container_id>:/home/postgres/db.csv
    ```
6. Import the CSV file into the Database
    ```SQL
    \copy journals (year, number, page, content)
    FROM '/home/postgres/db.csv'
    DELIMITER ',' CSV HEADER;
    ```
7. Add a Primary Key to the Table
    ```SQL
    ALTER TABLE journals ADD PRIMARY KEY (year, number, page);
    ```
8. Create the Vectoriser
    ```SQL
    SELECT ai.create_vectorizer(
    'journals'::regclass,  -- Specify the source table
    destination => 'journals_embeddings',  -- Specify the destination for embeddings
    embedding => ai.embedding_ollama('all-minilm', 384),  -- Use the embedding model (e.g., 'all-minilm' with 384 dimensions)
    chunking => ai.chunking_recursive_character_text_splitter('content')  -- Split the text in the 'content' column into chunks
    );
    ```

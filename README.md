
# Udiddit - A Social News Aggregator

Udiddit is a social news aggregator, content rating, and discussion platform. It allows registered users to link to external content or create their own posts on various topics—ranging from photography and food to more niche interests like horse masks or birds with arms. Users can engage with the content by commenting on posts and voting (up or down) to indicate their preferences.

## Project Overview

The current data model for Udiddit, stored in PostgreSQL, has shown limitations due to time constraints during the initial launch. This project addresses these issues through a series of steps aimed at redesigning the database schema, migrating the data, and implementing improvements to make the system more robust. The project also introduces web analytics to enhance the platform's capabilities.

## Project Structure

This project is divided into three main parts:

### Part I: Investigate the Existing Schema
- **Objective**: Review and assess the current database schema to identify issues and areas for improvement.
- **Approach**: SQL is utilized to query and explore the structure, relationships, and integrity of the existing data model.

### Part II: Create the DDL for the New Schema
- **Objective**: Design and implement a new database schema that resolves the flaws identified in the existing model.
- **Approach**: Using Data Definition Language (DDL) in SQL, a new, normalized schema is created. The design focuses on optimizing relationships, enforcing constraints, and improving overall efficiency.

### Part III: Migrate the Provided Data
- **Objective**: Migrate the existing data to the newly designed schema, ensuring data integrity and consistency.
- **Approach**: SQL scripts are written to transform and transfer the data from the old schema to the new schema, adhering to best practices for data migration.

## Requirements

- PostgreSQL for database management
- SQL knowledge to query, design, and manage database schemas
- Access to the existing Udiddit database (initial schema and data)
- SQL editor or development environment (e.g., pgAdmin, DBeaver)

## Installation

1. Set up the PostgreSQL environment on your system.
2. Access the existing schema and data.
3. Use a SQL editor to execute and manage SQL scripts.

## Usage

Follow the steps below to complete the project:

1. **Investigate the Existing Schema**:
   - Run the provided SQL queries to explore and analyze the current schema.
   - Document any issues and recommendations for improvement.

2. **Create the DDL for the New Schema**:
   - Write and execute DDL scripts to establish the new schema structure.
   - Ensure that all tables, constraints, and relationships are properly defined and normalized.

3. **Migrate the Data**:
   - Execute the migration scripts to transfer data from the old schema to the new one.
   - Verify the migration process for accuracy and consistency, ensuring no data loss.

## License

This project was done as part of the Udacity SQL Nano degree program.


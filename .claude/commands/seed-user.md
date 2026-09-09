description: create a single dummy uder in the databsase

allowed-tools:Read, Bash(python3:*)

Read database.py file to understand the users table 
schema and the get_db() helper.

Then write and run a Python script using Bash that:

1. Generates a realistic random indian user using your own knowledge of common Indian names across regions : 
    -Name: a realistic Indian first+last name 
    -Email: derived from the name with a random 2-3 digit number suffix (e.g. rahul.sharma91@gmail.com) 
    -Password: "password123" hashed with werkzeug's generate_password_hash
    -created_at: current datetime

2. Checks if the generated email already exists in the users table. If it does , regenerate until unique 

3. Insert the user into database using get_db() function 

4. Prints confirmation:
    - id
    - name 
    - email




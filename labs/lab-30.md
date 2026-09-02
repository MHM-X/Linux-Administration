# Lab 30: "Marrakech": Word Histogram

## Description
Find in the file frankestein.txt the second most frequent word and save in UPPER (capital) case in the /home/admin/mysolution file.

A word is a string of characters separated by space or newlines or punctuation symbols .,:; . Disregard case ('The', 'the' and 'THE' is the same word) and for simplification consider the apostrophe as another character not as punctuation ("it's" would be a word, distinct from "it" and "is"). Also disregard plurals ("car" and "cars" are different words) and other word variations (don't do "stemming").

We are providing a shorter test.txt file where the second most common word in upper case is "WORLD", so we could save this solution as: echo "WORLD" > /home/admin/mysolution

This problem can be done with a programming language (Python, Golang and sqlite3) or with common Linux utilities.

Test: echo "SOLUTION" | md5sum returns 19bf32b8725ec794d434280902d78e18

🔗 **Lab Link:** [SadServers - "Marrakech": Word Histogram](https://sadservers.com/scenario/marrakech)

<br>

## 🪜 Steps

### The solution is writing a bash script to find the required word 

```bash
#!/bin/bash

declare -A count

while IFS= read -r line; do

    # Replace the separators with spaces
    line=$(tr '.,:;' '    ' <<< "$line")

    for word in $line; do

        # Convert to lowercase
        word=${word,,}

        # Count the word
        current=${count["$word"]:-0}
        count["$word"]=$((current + 1))

    done

done < /home/admin/frankestein.txt


# Find the second most frequent word
second=$(for word in "${!count[@]}"; do
    printf '%s %s\n' "${count[$word]}" "$word"
done | sort -nr | sed -n '2p' | awk '{print $2}')


# Save the answer in uppercase
echo "${second^^}" > /home/admin/mysolution
```

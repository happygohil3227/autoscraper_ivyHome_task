## 1. Understanding the API Versions (v1, v2, v3)

The script interacts with an **autocomplete API** that provides search suggestions based on a given query. There are three different versions of the API, each allowing different types of characters and behaving slightly differently.

### 🔹 **v1 API (Basic Alphabet Search)**

- **Allowed characters**: Only **lowercase letters** (a-z).
- **Response behavior**: Returns up to **10 suggestions** per request.
- **Usage scenario**: Suitable for basic word predictions using only letters.
- **Example:**
  - Querying `"a"` might return:
    ```
    ['aa','aabdknlvkc','aabrkcd','aadgdqrwdy','aagqg','aaiha','aainmxg','aajfebume','aajwv','aakfubvxv']
    ```
  - Since the API returns the full **10 results**, it suggests there may be **more words starting with "a"**.
  - The script will then explore further, refining the search using prefixes like `"aal"`, `"aam"` to `"aaz"`.
  - Observation: The results are always in lexicographical order, meaning they are sorted alphabetically.

---

### 🔹 **v2 API (Alphanumeric Expansion)**

- **Allowed characters**: **Lowercase letters (a-z) + Numbers (0-9)**.
- **Response behavior**: Still limited to **12 suggestions per request**.
- **Usage scenario**: Useful for search terms that include numbers (e.g., product codes, usernames).
- **Example:**
  - Querying `"0"` could return:
    ```
    ['00981o7oyy','00muuu8','00o1z8b2t5','00tfan4','00us291vs','00vhuwj9','01','010uj5','013a6','01485vptaz','01iq', '01s0hi6']
    ```
  - Since numbers are included, the script must explore prefixes like `"02"`, `"03"`, `"04"`, etc.
  - Observation: The results are always in lexicographical order, meaning they are sorted alphabetically.

---

### 🔹 **v3 API (Expanded Character Set)**

- **Allowed characters**:
  - **Letters (a-z)**,
  - **Numbers (0-9)**,
  - **Spaces and special characters (**``**, **``**, **``**)**.
- **Response behavior**: Returns up to **15 suggestions** per request.
- **Usage scenario**: Suitable for more complex searches, including names, product IDs, and phrases.
- **Example:**
  - Querying `"0"` might return:
    ```
    ['0','0 .r m1','0 3','0 4','0 c.xcr+','0 u','0 v-v8gq', '0+22l2p8','0+d','0+e3ldrq', '0+h6i48r1j', '0+k94tv048','0+qcv-mazy','0+qy','0+yg39.ujr']
    ```
  - This version supports **spaces and symbols**, making search queries more flexible.

---

### 🚦 **Rate Limiting & Error Handling**

All three API versions enforce **rate limits**—meaning too many requests in a short period will trigger a **429 error (Too Many Requests)**. To handle this:

- If a **429 error** occurs, the script **pauses for a while (e.g., 20 seconds)** before retrying.
- The script **switches between different API keys** or **scraper services** to bypass limits when necessary.

---

### 🏁 **Summary of API Differences**

| API Version | Allowed Characters             | Max Suggestions |
| ----------- | ------------------------------ | --------------- |
| **v1**      | `a-z` (letters only)           | 10             |
| **v2**      | `a-z, 0-9` (letters + numbers) | 12              | 
| **v3**      | `a-z, 0-9, space, +, -, .`     | 15              |
This structure ensures efficient data collection while adapting to the API's behavior. 🚀

## 2. How It Works (Simple Explanation)

The script systematically queries the API to gather all possible autocomplete results. Here’s the approach:

1. **Start with the smallest valid character** (e.g., 'a' for v1, '0' for v2, or a space for v3).
2. **Ask the API for autocomplete suggestions** using the `fetch_results` function.
3. **Check how many results are returned**:
   - If we get the **maximum number of results** (12 for v1 & v2, 15 for v3), we assume there might be more words starting with this prefix.
   - We find the **longest common prefix** in the results and continue searching deeper.
4. **If fewer results are returned**, it means we’ve exhausted that prefix, so we move on to the next possible character.
5. **All results are saved** in text files (`list_1.txt`, `list_2.txt`, `list_3.txt`), making sure to avoid duplicates.

## 3. Key Functions Explained

### `fetch_results(query)`

- Sends a request to the API with a given query.
- If the API enforces a rate limit (429 error), it waits and retries.

### `find_common_characters(results)`

- Finds the longest shared prefix among the results, helping us decide the next search query.

### `next_c(common_characters)`

- Figures out the next search string by adding the next valid character.

## 4. The Main Code That Calls the APIs

```python
characters = v1_char[0]  # Start with 'a' for v1
api_requests_count = 0

dict_results = []
while True:
    results = fetch_results(characters)
    api_requests_count += 1
    
    with open('list_1.txt', 'a') as f:
        for i in results:
            if i not in dict_results:
                f.write("%s\n" % i)
    
    dict_results = results
    
    if len(results) == 12:
        common_characters = find_common_characters(results)
        common_characters += results[-1][len(common_characters)]
        characters = common_characters
    else:
        characters = next_c(characters)
        if characters == '-1':
            break

print(f'Total requests count is {api_requests_count}')
```

The same logic applies for **v2** and **v3**, but with different sets of characters (`v2_char`, `v3_char`).



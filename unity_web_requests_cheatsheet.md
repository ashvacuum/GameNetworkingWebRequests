# Unity Web Requests - Quick Reference Cheat Sheet

## Basic Template

```csharp
using UnityEngine;
using UnityEngine.Networking;
using System.Collections;

public class YourClass : MonoBehaviour
{
    IEnumerator YourWebRequest()
    {
        using (UnityWebRequest request = UnityWebRequest.Get("url"))
        {
            yield return request.SendWebRequest();
            
            if (request.result == UnityWebRequest.Result.Success)
            {
                // Handle success
            }
            else
            {
                // Handle error
            }
        }
    }
}
```

---

## GET Request (Retrieve Data)

```csharp
IEnumerator GetRequest(string url)
{
    using (UnityWebRequest request = UnityWebRequest.Get(url))
    {
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            string jsonResponse = request.downloadHandler.text;
            // Parse JSON: JsonUtility.FromJson<YourClass>(jsonResponse);
        }
    }
}
```

**Example URL:** `https://jsonplaceholder.typicode.com/posts/1`

---

## POST Request (Create Data)

```csharp
IEnumerator PostRequest(string url, string jsonData)
{
    using (UnityWebRequest request = new UnityWebRequest(url, "POST"))
    {
        byte[] bodyRaw = System.Text.Encoding.UTF8.GetBytes(jsonData);
        request.uploadHandler = new UploadHandlerRaw(bodyRaw);
        request.downloadHandler = new DownloadHandlerBuffer();
        request.SetRequestHeader("Content-Type", "application/json");
        
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log(request.downloadHandler.text);
        }
    }
}
```

**Example Usage:**
```csharp
string json = JsonUtility.ToJson(myObject);
StartCoroutine(PostRequest("https://jsonplaceholder.typicode.com/posts", json));
```

---

## PUT Request (Update Data)

```csharp
IEnumerator PutRequest(string url, string jsonData)
{
    using (UnityWebRequest request = new UnityWebRequest(url, "PUT"))
    {
        byte[] bodyRaw = System.Text.Encoding.UTF8.GetBytes(jsonData);
        request.uploadHandler = new UploadHandlerRaw(bodyRaw);
        request.downloadHandler = new DownloadHandlerBuffer();
        request.SetRequestHeader("Content-Type", "application/json");
        
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("Updated successfully");
        }
    }
}
```

**Example URL:** `https://jsonplaceholder.typicode.com/posts/1`

---

## DELETE Request (Remove Data)

```csharp
IEnumerator DeleteRequest(string url)
{
    using (UnityWebRequest request = UnityWebRequest.Delete(url))
    {
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("Deleted successfully");
        }
    }
}
```

**Example URL:** `https://jsonplaceholder.typicode.com/posts/1`

---

## JSON Parsing

### Single Object
```csharp
[System.Serializable]
public class Post
{
    public int id;
    public string title;
    public string body;
}

// Parse
Post post = JsonUtility.FromJson<Post>(jsonString);
```

### Array of Objects
```csharp
[System.Serializable]
public class PostList
{
    public Post[] posts;
}

// Wrap array in object
string wrapped = "{\"posts\":" + jsonString + "}";
PostList list = JsonUtility.FromJson<PostList>(wrapped);
```

---

## Error Handling

```csharp
if (request.result == UnityWebRequest.Result.ConnectionError)
{
    Debug.Log("No internet connection");
}
else if (request.result == UnityWebRequest.Result.ProtocolError)
{
    Debug.Log($"HTTP Error: {request.responseCode}");
}
else
{
    Debug.Log("Success!");
}
```

---

## Common Response Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 404 | Not Found |
| 500 | Server Error |

---

## Test APIs

### JSONPlaceholder (No Auth Required)
- **Base URL:** `https://jsonplaceholder.typicode.com`
- **Endpoints:**
  - `/posts` - 100 blog posts
  - `/users` - 10 users
  - `/comments` - 500 comments
  - `/todos` - 200 todo items

### Example Requests
```
GET    /posts          - Get all posts
GET    /posts/1        - Get post 1
GET    /posts?userId=1 - Get posts by user 1
POST   /posts          - Create new post
PUT    /posts/1        - Update post 1
DELETE /posts/1        - Delete post 1
```

---

## Tips & Best Practices

1. **Always use `using` statement** - Auto-disposes resources
2. **Set timeout** - `request.timeout = 10;`
3. **Check result type** - Don't just check for errors, check success too
4. **Use coroutines** - Never block the main thread
5. **Set Content-Type** - Required for POST/PUT with JSON
6. **Handle errors gracefully** - Show user-friendly messages

---

## Common Mistakes

❌ **Forgetting Content-Type header**
```csharp
// Missing header for JSON!
request.uploadHandler = new UploadHandlerRaw(bodyRaw);
```

✅ **Correct**
```csharp
request.uploadHandler = new UploadHandlerRaw(bodyRaw);
request.SetRequestHeader("Content-Type", "application/json");
```

---

❌ **Not using `using` statement**
```csharp
UnityWebRequest request = UnityWebRequest.Get(url);
// Forgot to dispose!
```

✅ **Correct**
```csharp
using (UnityWebRequest request = UnityWebRequest.Get(url))
{
    // Auto-disposed when block exits
}
```

---

❌ **Parsing arrays directly**
```csharp
Post[] posts = JsonUtility.FromJson<Post[]>(jsonString); // ERROR!
```

✅ **Correct**
```csharp
string wrapped = "{\"posts\":" + jsonString + "}";
PostList list = JsonUtility.FromJson<PostList>(wrapped);
Post[] posts = list.posts;
```

---

## Quick Start Checklist

- [ ] Import `UnityEngine.Networking`
- [ ] Use `IEnumerator` for web requests
- [ ] Call with `StartCoroutine()`
- [ ] Use `using` statement
- [ ] Check `request.result`
- [ ] Handle both success and error cases
- [ ] Set appropriate headers for POST/PUT
- [ ] Parse JSON with `JsonUtility`
- [ ] Create `[Serializable]` classes for JSON

---

## Need Help?

**Unity Documentation:**
- UnityWebRequest Manual: https://docs.unity3d.com/Manual/UnityWebRequest.html
- Scripting API: https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html

**Test Your Code:**
- JSONPlaceholder: https://jsonplaceholder.typicode.com
- Postman: https://www.postman.com (for testing APIs)
- JSON Formatter: https://jsonformatter.org (validate JSON)

# Unity Web Requests - Complete Lesson

## Learning Objectives
By the end of this lesson, students will be able to:
- Understand the different types of Unity Web Requests
- Implement GET, POST, PUT, and DELETE requests
- Handle JSON data from APIs
- Implement proper error handling
- Work with real-world API endpoints

---

## What is UnityWebRequest?

`UnityWebRequest` is Unity's modern system for making HTTP/HTTPS requests. It replaced the older `WWW` class and provides better memory management, more control, and support for various content types.

### Why Use UnityWebRequest?
- **Non-blocking**: Uses coroutines to avoid freezing your game
- **Memory efficient**: Better handling of large files
- **Flexible**: Supports all common HTTP methods
- **Cross-platform**: Works across all Unity platforms

---

## Types of Web Requests

### 1. GET Request
**Purpose**: Retrieve data from a server

**When to use**:
- Fetching user profiles
- Getting game leaderboards
- Retrieving game configuration
- Loading dynamic content

### 2. POST Request
**Purpose**: Send data to create new resources

**When to use**:
- Creating new user accounts
- Submitting high scores
- Uploading player data
- Sending analytics events

### 3. PUT Request
**Purpose**: Update existing resources

**When to use**:
- Updating user profiles
- Modifying game settings
- Changing player stats

### 4. DELETE Request
**Purpose**: Remove resources from server

**When to use**:
- Deleting user accounts
- Removing saved games
- Clearing cache data

---

## Sample API for Testing

We'll use **JSONPlaceholder** - a free fake REST API for testing:
- Base URL: `https://jsonplaceholder.typicode.com`
- No authentication required
- Perfect for learning and prototyping

### Available Endpoints:
- `/posts` - Blog posts (100 items)
- `/users` - User data (10 items)
- `/comments` - Comments (500 items)
- `/todos` - Todo items (200 items)

---

## Implementation Examples

### Basic Setup

First, create a `WebRequestManager` script:

```csharp
using UnityEngine;
using UnityEngine.Networking;
using System.Collections;
using System.Collections.Generic;

public class WebRequestManager : MonoBehaviour
{
    private const string BASE_URL = "https://jsonplaceholder.typicode.com";
    
    void Start()
    {
        // Test all request types
        StartCoroutine(GetExample());
        StartCoroutine(PostExample());
        StartCoroutine(PutExample());
        StartCoroutine(DeleteExample());
    }
}
```

---

### 1. GET Request - Retrieve Data

```csharp
IEnumerator GetExample()
{
    string url = BASE_URL + "/posts/1";
    
    using (UnityWebRequest request = UnityWebRequest.Get(url))
    {
        // Send the request
        yield return request.SendWebRequest();
        
        // Check for errors
        if (request.result == UnityWebRequest.Result.ConnectionError || 
            request.result == UnityWebRequest.Result.ProtocolError)
        {
            Debug.LogError("GET Error: " + request.error);
        }
        else
        {
            // Success - parse the response
            Debug.Log("GET Response: " + request.downloadHandler.text);
            
            // Parse JSON
            Post post = JsonUtility.FromJson<Post>(request.downloadHandler.text);
            Debug.Log($"Post Title: {post.title}");
        }
    }
}

// Data class for JSON parsing
[System.Serializable]
public class Post
{
    public int userId;
    public int id;
    public string title;
    public string body;
}
```

**GET All Posts Example:**
```csharp
IEnumerator GetAllPosts()
{
    string url = BASE_URL + "/posts";
    
    using (UnityWebRequest request = UnityWebRequest.Get(url))
    {
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            // For arrays, wrap in a container class
            string jsonArray = "{\"posts\":" + request.downloadHandler.text + "}";
            PostList postList = JsonUtility.FromJson<PostList>(jsonArray);
            
            Debug.Log($"Received {postList.posts.Length} posts");
            foreach (Post post in postList.posts)
            {
                Debug.Log($"- {post.title}");
            }
        }
    }
}

[System.Serializable]
public class PostList
{
    public Post[] posts;
}
```

---

### 2. POST Request - Create New Data

```csharp
IEnumerator PostExample()
{
    string url = BASE_URL + "/posts";
    
    // Create data to send
    Post newPost = new Post
    {
        userId = 1,
        title = "My Awesome Game",
        body = "This is a post created from Unity!"
    };
    
    // Convert to JSON
    string jsonData = JsonUtility.ToJson(newPost);
    
    using (UnityWebRequest request = new UnityWebRequest(url, "POST"))
    {
        // Attach JSON data
        byte[] bodyRaw = System.Text.Encoding.UTF8.GetBytes(jsonData);
        request.uploadHandler = new UploadHandlerRaw(bodyRaw);
        request.downloadHandler = new DownloadHandlerBuffer();
        
        // Set headers
        request.SetRequestHeader("Content-Type", "application/json");
        
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("POST Success: " + request.downloadHandler.text);
            
            // Server returns the created post with an ID
            Post createdPost = JsonUtility.FromJson<Post>(request.downloadHandler.text);
            Debug.Log($"Created post with ID: {createdPost.id}");
        }
        else
        {
            Debug.LogError("POST Error: " + request.error);
        }
    }
}
```

**Alternative POST Method:**
```csharp
IEnumerator PostWithForm()
{
    string url = BASE_URL + "/posts";
    
    // Using WWWForm (easier for simple data)
    WWWForm form = new WWWForm();
    form.AddField("userId", 1);
    form.AddField("title", "Form Post");
    form.AddField("body", "This uses WWWForm");
    
    using (UnityWebRequest request = UnityWebRequest.Post(url, form))
    {
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("Form POST Success: " + request.downloadHandler.text);
        }
    }
}
```

---

### 3. PUT Request - Update Existing Data

```csharp
IEnumerator PutExample()
{
    // Update post with ID 1
    string url = BASE_URL + "/posts/1";
    
    // Updated data
    Post updatedPost = new Post
    {
        userId = 1,
        id = 1,
        title = "Updated Title from Unity",
        body = "This post has been updated!"
    };
    
    string jsonData = JsonUtility.ToJson(updatedPost);
    
    using (UnityWebRequest request = new UnityWebRequest(url, "PUT"))
    {
        byte[] bodyRaw = System.Text.Encoding.UTF8.GetBytes(jsonData);
        request.uploadHandler = new UploadHandlerRaw(bodyRaw);
        request.downloadHandler = new DownloadHandlerBuffer();
        request.SetRequestHeader("Content-Type", "application/json");
        
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("PUT Success: " + request.downloadHandler.text);
        }
        else
        {
            Debug.LogError("PUT Error: " + request.error);
        }
    }
}
```

---

### 4. DELETE Request - Remove Data

```csharp
IEnumerator DeleteExample()
{
    // Delete post with ID 1
    string url = BASE_URL + "/posts/1";
    
    using (UnityWebRequest request = UnityWebRequest.Delete(url))
    {
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("DELETE Success - Post removed");
            Debug.Log("Response: " + request.downloadHandler.text);
        }
        else
        {
            Debug.LogError("DELETE Error: " + request.error);
        }
    }
}
```

---

## Advanced Topics

### Handling Authentication (Bearer Tokens)

```csharp
IEnumerator AuthenticatedRequest(string token)
{
    string url = "https://your-api.com/protected-endpoint";
    
    using (UnityWebRequest request = UnityWebRequest.Get(url))
    {
        request.SetRequestHeader("Authorization", "Bearer " + token);
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("Authenticated request successful");
        }
    }
}
```

### Download Progress Tracking

```csharp
IEnumerator DownloadWithProgress(string url)
{
    using (UnityWebRequest request = UnityWebRequest.Get(url))
    {
        var operation = request.SendWebRequest();
        
        while (!operation.isDone)
        {
            float progress = request.downloadProgress * 100f;
            Debug.Log($"Download Progress: {progress:F2}%");
            yield return null;
        }
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            Debug.Log("Download complete!");
        }
    }
}
```

### Timeout Handling

```csharp
IEnumerator RequestWithTimeout(string url, int timeoutSeconds = 10)
{
    using (UnityWebRequest request = UnityWebRequest.Get(url))
    {
        request.timeout = timeoutSeconds;
        
        yield return request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.ConnectionError)
        {
            Debug.LogError("Request timed out or connection failed");
        }
    }
}
```

---

## Best Practices

### 1. Always Use `using` Statement
```csharp
// Good - automatically disposes resources
using (UnityWebRequest request = UnityWebRequest.Get(url))
{
    yield return request.SendWebRequest();
}

// Bad - manual disposal required
UnityWebRequest request = UnityWebRequest.Get(url);
yield return request.SendWebRequest();
request.Dispose(); // Easy to forget!
```

### 2. Check Result Types
```csharp
// Check for specific error types
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

### 3. Use Async/Await (Unity 2023+)
```csharp
using System.Threading.Tasks;

async Task<string> GetDataAsync()
{
    string url = BASE_URL + "/posts/1";
    
    using (UnityWebRequest request = UnityWebRequest.Get(url))
    {
        await request.SendWebRequest();
        
        if (request.result == UnityWebRequest.Result.Success)
        {
            return request.downloadHandler.text;
        }
        return null;
    }
}
```

---

## Common Error Codes

| Code | Meaning |
|------|---------|
| 200 | OK - Request succeeded |
| 201 | Created - Resource created successfully |
| 400 | Bad Request - Invalid data sent |
| 401 | Unauthorized - Authentication required |
| 404 | Not Found - Resource doesn't exist |
| 500 | Internal Server Error - Server problem |

---

## Practice Exercises

### Exercise 1: User Manager
Create a script that can:
- GET all users from `/users`
- Display user names in the console
- Count how many users live in a specific city

### Exercise 2: Todo List
Build a simple todo system:
- GET all todos from `/todos`
- POST a new todo
- PUT to update a todo's completion status
- DELETE a completed todo

### Exercise 3: Comment System
Create a comment viewer:
- GET comments for a specific post ID
- Filter comments by email domain
- Display the total character count of all comments

### Exercise 4: Error Handling
Implement robust error handling:
- Test with invalid URLs
- Handle timeout scenarios
- Retry failed requests (max 3 attempts)
- Show user-friendly error messages

---

## Additional Resources

### Other Free APIs for Practice:
1. **PokéAPI**: `https://pokeapi.co/api/v2/`
   - Pokemon data, types, abilities
   
2. **OpenWeatherMap**: `https://openweathermap.org/api`
   - Weather data (requires free API key)
   
3. **REST Countries**: `https://restcountries.com/v3.1/`
   - Country information and flags
   
4. **The Cat API**: `https://api.thecatapi.com/v1/`
   - Random cat images (fun for testing!)

### Unity Documentation:
- [UnityWebRequest Manual](https://docs.unity3d.com/Manual/UnityWebRequest.html)
- [UnityWebRequest Scripting API](https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html)

---

## Summary

**Key Takeaways:**
- Use `UnityWebRequest` for all HTTP operations in Unity
- Always use coroutines or async/await for non-blocking requests
- Remember to set appropriate headers for JSON data
- Implement proper error handling for production code
- Use the `using` statement to ensure proper resource disposal
- Test with free APIs like JSONPlaceholder before implementing real APIs

**Common Pitfalls to Avoid:**
- Forgetting to set Content-Type header for POST/PUT
- Not handling errors properly
- Making requests on the main thread (blocking)
- Forgetting to dispose of UnityWebRequest objects
- Not checking responseCode for HTTP errors

---

## Quiz Questions

1. What is the main advantage of UnityWebRequest over the old WWW class?
2. Which HTTP method would you use to create a new user account?
3. What does the `using` statement do in a UnityWebRequest?
4. How do you check if a request completed successfully?
5. What header must you set when sending JSON data?

**Answers:**
1. Better memory management, more control, support for all HTTP methods
2. POST
3. Automatically disposes of resources when the block exits
4. Check if `request.result == UnityWebRequest.Result.Success`
5. `Content-Type: application/json`

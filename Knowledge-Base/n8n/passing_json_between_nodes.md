# n8n: Passing JSON Between Nodes Without Breaking the Payload

## What I Learned

While building an AI-powered blog automation workflow, I ran into a confusing JSON error.

My workflow looked roughly like:

```text
AI Agent → Code Node → HTTP Request → Ghost API
```

The AI Agent generated the blog content, the Code node converted it into Ghost's Lexical format, and the HTTP Request node sent the payload to Ghost.

The problem appeared when the HTTP Request node reported:

```text
JSON parameter needs to be valid JSON
```

At first, I suspected:

* The AI model was hallucinating.
* The Ghost API was rejecting the Lexical structure.
* The Code node was generating invalid data.
* The parser was broken.

The actual problem was simpler: **I was unnecessarily rebuilding JSON that the previous node had already created.**

---

## Understanding `$json`

In n8n:

```javascript
$json
```

represents the **JSON data of the current item**.

For example, if a node outputs:

```json
{
  "student": [
    {
      "name": "Rakin"
    },
    {
      "name": "Monica"
    }
  ]
}
```

Then:

```javascript
$json.student[0].name
```

returns:

```text
Rakin
```

The breakdown is:

```text
$json
  └── student
       └── [0]
            └── name
```

Similarly:

```javascript
$json.student[1].name
```

returns:

```text
Monica
```

---

## The Mistake

My Code node had already produced a complete object:

```json
{
  "posts": [
    {
      "title": "...",
      "slug": "...",
      "lexical": "...",
      "status": "published"
    }
  ]
}
```

But in the HTTP Request node, I tried to manually reconstruct it:

```json
{
  "posts": [
    {
      "title": "{{ $json.posts[0].title }}",
      "lexical": "{{ $json.posts[0].lexical }}",
      "status": "published"
    }
  ]
}
```

This was unnecessary and caused problems because `lexical` was itself a **serialized JSON string**.

In other words:

```text
JSON object
   ↓
posts
   ↓
lexical
   ↓
JSON stored inside a string
```

Manually inserting that serialized JSON into another JSON structure can create complicated escaping and quoting problems.

---

## The Better Approach

If the previous node has already created the exact payload required by the API, don't reconstruct it.

Pass the entire current JSON object:

```javascript
{{ $json }}
```

The HTTP Request node can then receive the object produced by the Code node directly.

Conceptually:

```text
Code Node
   │
   │ produces complete JSON object
   ▼
$json
   │
   │ pass directly
   ▼
HTTP Request
   │
   ▼
Ghost API
```

This worked because I stopped unnecessarily transforming the data between the Code node and the HTTP Request node.

---

## Important Distinction: Object vs JSON String

This is one of the most important things I learned.

These are not the same:

### JavaScript/JSON object

```javascript
{
  title: "My Blog"
}
```

### JSON string

```javascript
"{\"title\":\"My Blog\"}"
```

The second one is **text containing JSON**.

JavaScript provides two useful functions for converting between them:

```javascript
JSON.stringify(object)
```

Converts an object → JSON string.

And:

```javascript
JSON.parse(string)
```

Converts a JSON string → JavaScript object.

For example:

```javascript
const object = {
  title: "My Blog"
};

const string = JSON.stringify(object);

const objectAgain = JSON.parse(string);
```

So when debugging n8n, I should always ask:

> **Am I dealing with an object, or a string containing JSON?**

---

## Debugging Lesson

When an API reports:

```text
JSON parameter needs to be valid JSON
```

don't immediately assume the API is broken.

Inspect the data entering the HTTP Request node.

A useful debugging process is:

```text
1. Look at the previous node's output.
2. Identify the exact JSON structure.
3. Check whether values are objects, arrays, or strings.
4. Check whether JSON has been serialized with JSON.stringify().
5. Avoid rebuilding data unnecessarily.
6. Pass the existing object directly when possible.
```

---

## Key Takeaway

The biggest lesson wasn't about Ghost or Lexical.

It was about **following the data**.

Instead of thinking:

> "Which node is broken?"

I should think:

> "What exact data is coming out of this node, and what exact data is going into the next one?"

Once I started treating the workflow as a chain of data transformations, the problem became much easier to understand.

### Rule to remember

**If a previous node already produced the complete API payload, don't manually rebuild it unless you actually need to transform it.**

Sometimes the complicated AI automation bug is simply:

```text
JSON → JSON inside a string → JSON → 💥
```

And sometimes the solution is just:

```javascript
{{ $json }}
```

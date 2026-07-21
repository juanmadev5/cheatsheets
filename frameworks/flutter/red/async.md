# Flutter — Async (FutureBuilder / StreamBuilder)

---

## 🌊 Manejo de async — FutureBuilder y StreamBuilder

```dart
FutureBuilder<List<Post>>(
  future: ApiService().fetchPosts(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return const CircularProgressIndicator();
    }
    if (snapshot.hasError) return Text('Error: ${snapshot.error}');
    if (!snapshot.hasData || snapshot.data!.isEmpty) return const Text('Sin datos');

    final posts = snapshot.data!;
    return ListView.builder(
      itemCount: posts.length,
      itemBuilder: (_, i) => ListTile(title: Text(posts[i].title)),
    );
  },
)

StreamBuilder<int>(
  stream: timerStream,
  initialData: 0,
  builder: (context, snapshot) {
    return Text('${snapshot.data}');
  },
)
```



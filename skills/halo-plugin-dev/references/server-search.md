# Search Integration

Integrate custom extensions with Halo's built-in search engine.

> Source: [HaloDocument](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/search/HaloDocument.java) | [HaloDocumentsProvider](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/search/HaloDocumentsProvider.java) | [SearchEngine](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/search/SearchEngine.java)

## HaloDocumentsProvider

Implement this extension point to make your custom Extension searchable.

```java
@Component
public class MyDocumentProvider implements HaloDocumentsProvider {

    private final ReactiveExtensionClient client;

    public MyDocumentProvider(ReactiveExtensionClient client) {
        this.client = client;
    }

    @Override
    public Flux<HaloDocument> fetchAll() {
        return client.listAll(MyExtension.class, new ListOptions(), Sort.unsorted())
            .filter(ext -> ext.getMetadata().getDeletionTimestamp() == null)
            .map(this::convertToDocument);
    }

    @Override
    public String getType() {
        return "myextension.mygroup.halo.run";
    }

    private HaloDocument convertToDocument(MyExtension ext) {
        var doc = new HaloDocument();
        doc.setId(ext.getMetadata().getName());
        doc.setMetadataName(ext.getMetadata().getName());
        doc.setTitle(ext.getSpec().getTitle());
        doc.setDescription(ext.getSpec().getDescription());
        doc.setContent(stripHtml(ext.getSpec().getContent()));
        doc.setOwnerName(ext.getSpec().getOwner());
        doc.setCreationTimestamp(ext.getMetadata().getCreationTimestamp());
        doc.setUpdateTimestamp(ext.getMetadata().getCreationTimestamp());
        doc.setPermalink("/my-extensions/" + ext.getSpec().getSlug());
        doc.setType(getType());
        doc.setPublished(true);
        doc.setRecycled(false);
        doc.setExposed(true);
        return doc;
    }
}
```

## Search Events

Publish events to add/remove documents from the search index:

```java
// Add or update documents
applicationEventPublisher.publishEvent(
    new HaloDocumentAddRequestEvent(this, List.of(haloDoc)));

// Delete documents by ID
applicationEventPublisher.publishEvent(
    new HaloDocumentDeleteRequestEvent(this, List.of("doc-id-1", "doc-id-2")));

// Rebuild entire index (e.g., on plugin config change)
applicationEventPublisher.publishEvent(
    new HaloDocumentRebuildRequestEvent(this));
```

## SearchEngine (Replacing Default)

To replace Halo's default search engine (e.g., with Meilisearch):

```java
@Component
public class MeilisearchEngine implements SearchEngine {

    @Override
    public boolean available() {
        return meilisearchClient != null && meilisearchClient.isHealthy();
    }

    @Override
    public void addOrUpdate(Iterable<HaloDocument> docs) {
        // Batch index documents
    }

    @Override
    public void deleteDocument(Iterable<String> ids) {
        // Batch delete by ID
    }

    @Override
    public void deleteAll() {
        // Clear index
    }

    @Override
    public SearchResult search(SearchOption option) {
        // Execute search query
    }
}
```

## HaloDocument Fields

| Field                                   | Description                  | Required |
| --------------------------------------- | ---------------------------- | -------- |
| `id`                                    | Global unique document ID    | Yes      |
| `metadataName`                          | Extension metadata name      | Yes      |
| `title`                                 | Document title               | Yes      |
| `content`                               | Plain text content (no HTML) | Yes      |
| `description`                           | Short description            | No       |
| `permalink`                             | URL path                     | Yes      |
| `type`                                  | Document type identifier     | Yes      |
| `ownerName`                             | Owner metadata name          | Yes      |
| `categories`                            | Category metadata names      | No       |
| `tags`                                  | Tag metadata names           | No       |
| `published`                             | Whether published            | Yes      |
| `recycled`                              | Whether in recycle bin       | Yes      |
| `exposed`                               | Whether publicly visible     | Yes      |
| `creationTimestamp` / `updateTimestamp` | Timestamps                   | Yes      |

# Postgram
> An instagram clone app for learning purposes, to be continued...

----

### Table of Contents

- [Description](#description)
- [Technologies](#technologies)
- [Usage](#usage)
- [Author Info](#author-info)

---

### Description

An instagram clone app for learning purposes, written in Swift (with usage of UIKit) on the client side and Node.js (with usage of Javascript) in backend. Design modeled on current social app. Idea inspired by the social app courses on internet and recreated by my own. App data is stored and maintained in new NoSQL Cloud Firestroe db. Database is supported by Cloud Functions that provide serverless computing. The app is still being in development.

### Technologies

Written in Swift UIKit in programmatically approach with usage of:
- [Cloud Firestore](https://firebase.google.com/docs/firestore)
- [SDWebImage](https://github.com/SDWebImage/SDWebImage)
- [Cloud Functions](https://firebase.google.com/docs/functions) in [Node Enviroment](https://nodejs.org/en/about/)


### Usage

<!---!![](project-image-url)--->
<!---![GoogleSheets sheet](https://j.gifs.com/QnozZG.gif)--->
<img src="https://j.gifs.com/ANB8z1.gif" width="233" height="481"/> 
<!---!<div class="row">
  <div class="column">
    <img src="" width="233" height="481"/>
  </div>
  <div class="column">
    <img src="" width="233" height="481"/>
  </div>
</div>--->


### Code snippets

#### - Pagination
In Service class we run our query, then we follow last document we fetched in previous batch of data to fetch next batch after it.
```
static func fetchPostWithLimit(limit: Int, isNextFetch: Bool, documents: [QueryDocumentSnapshot], completion: @escaping([Post], [QueryDocumentSnapshot]) -> Void) {
    guard let uid = Auth.auth().currentUser?.uid else { return }
    var posts = [Post]()
    
    var query = COLLECTION_USERS.document(uid).collection("user-feed").limit(to: limit).order(by: "timestamp", descending: true)
    
    if isNextFetch == true {
        guard let lastDocument = documents.last else { return }
        query = query.start(afterDocument: lastDocument)
    }
    
    query.getDocuments { snapshot, error in
        guard let documents = snapshot?.documents else { return }
        documents.forEach({ document in
            fetchPost(withPostId: document.documentID) { post in
                posts.append(post)
                posts.sort(by: { $0.timestamp.seconds > $1.timestamp.seconds })
                if posts.count == documents.count { 
                    completion(posts, documents)
                }
            }
        })
    }
}    
```
In ViewController reponsible for feed first of all we store temporary documents from previous fetch of data.
```
var documents = [QueryDocumentSnapshot]()
```
function called in viewDidLoad() after the view controller has loaded its view hierarchy into memory.
```
func fetchPosts() {
    guard post == nil else { return }
    
    PostService.fetchPostWithLimit(limit: 5, isNextFetch: false, documents: documents) { (posts, documents) in
        self.posts = posts
        self.documents = documents
        self.checkIfUserLikedPost()
    }
}
```
Trigger pagination e.g when user scrolled to last item
```
override func collectionView(_ collectionView: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt indexPath: IndexPath) {
    let lastItem = self.posts.count - 1
    if (indexPath.item == lastItem) {
        if indexPath.item > 1 {
            PostService.fetchPostWithLimit(limit: 4, isNextFetch: true, documents: documents) { (posts, documents) in
                self.posts.append(contentsOf: posts)
                self.documents = documents
                self.checkIfUserLikedPost()
                self.collectionView.refreshControl?.endRefreshing()
            }
        }
    }
}
```


---

#### Author Info

- Github - [@szytambsky](https://github.com/szytambsky)
- Pictures - All credit goes to the artist from unsplash.com

[Back To The Top](#smart-structures-support)

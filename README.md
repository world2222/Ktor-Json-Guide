# Ktor-Json-Guide
You can learn how to get values from Json files using Ktor.  
<br>

## Before you read...
It took legit a lot of time to find out how it works with ktor.  

There were not that many infos about it and it has a bit recently updated so the things that most of the Youtubers were talking about with this feature were all old and some of their codes were not even available to use for now.  

So based on the code that a Youtuber has made, I changed some codes to the newest version and after about 7 hours it finally worked..!!  
<br>

## Just remember these things then you can use ktor
### 1. Use these dependencies
```kotlin
// Put these in build.grade APP file.
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
    id 'dagger.hilt.android.plugin'
    id 'org.jetbrains.kotlin.plugin.serialization' version '1.9.0'
}

dependencies {
    // Ktor
    def ktor_version = "2.3.2"
    implementation "io.ktor:ktor-client-core:$ktor_version"
    implementation "io.ktor:ktor-client-android:$ktor_version"
    implementation "io.ktor:ktor-client-content-negotiation:$ktor_version"
    implementation "io.ktor:ktor-client-logging:$ktor_version"
    implementation "io.ktor:ktor-serialization-kotlinx-json:$ktor_version"
    
    // Serialization
    implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.5.1'
}
```
<br>

```kotlin
// Put this in build.gradle PROJECT file.
id 'org.jetbrains.kotlin.android' version '1.9.0' apply false
```
<br>

### 2. Create 'PostResponse' and 'PostRequest' files in data.remote.dto
```kotlin
// In PostResponse.kt
@Serializable        // Essential to let it knows that it should have to be Serializable
data class PostResponse(
    val userId: Int,
    val id: Int,
    val title: String,
    val body: String
)

// In PostRequest.kt
@Serializable        // No id here cuz it comes from the server
data class PostRequest(
    val userId: Int,
    val title: String,
    val body: String
)
```
<br>

### 3. Create 'PostService' interface in data.remote
```kotlin
interface PostsService {

    // For getting posts. Return as List<PostResponse> type.
    suspend fun getPosts(): List<PostResponse>

    // For creating posts.
    // 'postRequest' is the data we serialize to json and send to the server.
    // Return as PostResponse and it's nullable cuz it could go wrong so put '?' at the end.
    suspend fun createPost(postRequest: PostRequest): PostResponse?

    // These codes should be written after writing codes on no.4
    companion object {
        fun create(): PostsService {
            return PostsServiceImpl(
                client = HttpClient(Android) {
                    install(ContentNegotiation) {
                        json()
                    }
                }
            )
        }
    }
}
```
<br>

### 4. Create 'PostsServiceImpl' file in data.remote
'PostsServiceImpl' file is simply a file that implements features in 'PostsService' interface.
```kotlin
class PostsServiceImpl(
    private val client: HttpClient
): PostsService {

    // Overriding the function getPosts() describing how to get data from the api
    override suspend fun getPosts(): List<PostResponse> {

        return try {
            client.get(HttpRoutes.POSTS).body()    // Tries to receive the payload of the response as a specific type T. Here, Json type.
        } catch (e: RedirectResponseException) {
            // 3xx - responses
            println("Error: ${e.response.status.description}")
            emptyList()
        } catch (e: ClientRequestException) {
            // 4xx - responses
            println("Error: ${e.response.status.description}")
            emptyList()
        } catch (e: ServerResponseException) {
            // 5xx - responses
            println("Error: ${e.response.status.description}")
            emptyList()
        } catch (e: Exception) {
            // 3xx - responses
            println("Error: ${e.message}")
            emptyList()
        }
    }

    // Overriding the function createPost() describing how to send data to api
    override suspend fun createPost(postRequest: PostRequest): PostResponse? {

        return try {
            client.post(HttpRoutes.POSTS).body()
        } catch (e: RedirectResponseException) {
            // 3xx - responses
            println("Error: ${e.response.status.description}")
            null
        } catch (e: ClientRequestException) {
            // 4xx - responses
            println("Error: ${e.response.status.description}")
            null
        } catch (e: ServerResponseException) {
            // 5xx - responses
            println("Error: ${e.response.status.description}")
            null
        } catch (e: Exception) {
            // 3xx - responses
            println("Error: ${e.message}")
            null
        }
    }
}
```
<br>

### 5. Create values in 'MainActivity' file
```kotlin
class MainActivity : ComponentActivity() {

    // Create a value for PostsServiceImpl so that we can use the features of PostsService.
    private val service = PostsService.create()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {

            // Create a value using produceState so that we can use the value in coroutine.
            // Cuz the api related calls should have to be implemented asynchronously, It's essential to use coroutine.
            val posts = produceState<List<PostResponse>>(
                initialValue = emptyList(),
                producer = {
                    value = service.getPosts()
                }
            )

            KtorGuideTheme {
                LazyColumn {
                    // Applying the same way to implement with each element from json.
                    items(posts.value) {
                        Column(
                            modifier = Modifier
                                .fillMaxWidth()
                                .padding(16.dp)
                        ) {
                            Text(text = it.title, fontSize = 20.sp)
                            Spacer(modifier = Modifier.height(4.dp))
                            Text(text = it.body, fontSize = 14.sp)
                        }
                    }
                }
            }
        }
    }
}
```
***
As I remember, using retrofit was extremely so hard cuz it was too complicated.  
Maybe cuz of that reason, Ktor feels way easier and efficient for me.  
<br>
So from now on, I'll use Ktor as possible as I can with every project.  
And feel so proud to build my own app using api call without any helps.

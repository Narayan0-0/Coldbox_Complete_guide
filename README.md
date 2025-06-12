# ColdBox Framework: Complete Step-by-Step Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Project Structure](#project-structure)
4. [Configuration](#configuration)
5. [Your First Handler](#your-first-handler)
6. [Views and Layouts](#views-and-layouts)
7. [Models and Services](#models-and-services)
8. [Database Integration](#database-integration)
9. [Routing](#routing)
10. [Testing](#testing)
11. [Advanced Features](#advanced-features)

## Prerequisites

Before starting with ColdBox, ensure you have:
- **CFML Engine**: Adobe ColdFusion 2018+ or Lucee 5.3+
- **CommandBox**: ColdBox's CLI tool for development
- **Web Server**: Apache, IIS, or built-in server
- **Database**: MySQL, PostgreSQL, SQL Server, or similar (optional)

  
(NOTE: Plase ensure that you download the latest version for it. These versions as per JUNE 2025)

## Installation

### Step 1: Install CommandBox

Download and install CommandBox from [https://www.ortussolutions.com/products/commandbox](https://www.ortussolutions.com/products/commandbox)

```bash
# Verify installation
box version
```

### Step 2: Create a New ColdBox Application

```bash
# Create a new ColdBox application
mkdir my-coldbox-app
cd my-coldbox-app

# Initialize ColdBox application
box coldbox create app MyApp

# Or use the template approach
box coldbox create app name=MyApp skeleton=AdvancedScript
```

### Step 3: Start the Development Server

```bash
# Start the embedded server
box server start

# Your app will be available at http://127.0.0.1:8080
```

## Project Structure

After installation, your ColdBox application will have this structure:

```
MyApp/
├── coldbox/                 # ColdBox framework files
├── config/                  # Configuration files
│   ├── Coldbox.cfc         # Main ColdBox configuration
│   ├── Router.cfc          # URL routing configuration
│   └── WireBox.cfc         # Dependency injection configuration
├── handlers/               # Controllers (MVC Controllers)
│   └── Main.cfc           # Default handler
├── interceptors/          # Event interceptors
├── layouts/               # View layouts
│   └── Main.cfm          # Default layout
├── models/                # Business logic models
├── modules/               # ColdBox modules
├── modules_app/           # Application-specific modules
├── views/                 # Views (MVC Views)
│   └── main/             # Views for Main handler
│       ├── index.cfm     # Default view
│       └── error.cfm     # Error view
├── includes/              # Static assets (CSS, JS, images)
├── tests/                 # Unit and integration tests
├── Application.cfc        # ColdFusion application settings
├── index.cfm             # Application entry point
└── box.json              # CommandBox package descriptor
```

## Configuration

### Step 4: Configure ColdBox Settings

Edit `config/Coldbox.cfc`:

```javascript
component {
    
    function configure() {
        
        // ColdBox Configuration
        coldbox = {
            // Application name
            appName = "MyApp",
            
            // Default event to execute
            defaultEvent = "main.index",
            
            // Default layout
            defaultLayout = "Main.cfm",
            
            // Development settings
            reinitPassword = "1234",
            handlersIndexAutoReload = true,
            
            // Implicit event settings
            implicitViews = true,
            implicitViewChecks = true,
            
            // Error handling
            exceptionHandler = "main.onException",
            onInvalidEvent = "main.onInvalidEvent",
            
            // Extension points
            applicationStartHandler = "main.onAppInit",
            applicationEndHandler = "main.onAppEnd",
            sessionStartHandler = "main.onSessionStart",
            sessionEndHandler = "main.onSessionEnd"
        };
        
        // Environment settings
        environments = {
            development = "localhost,127.0.0.1",
            production = "www.myapp.com"
        };
        
        // Module settings
        moduleSettings = {};
        
        // Interceptor settings
        interceptorSettings = {
            throwOnInvalidStates = false,
            customInterceptionPoints = []
        };
        
        // Logging settings
        logBox = {
            appenders = {
                coldboxTracer = { class="coldbox.system.logging.appenders.ConsoleAppender" }
            },
            root = { levelMin="FATAL", levelMax="DEBUG", appenders="*" }
        };
        
        // Layout settings
        layoutSettings = {
            defaultLayout = "Main.cfm"
        };
        
        // i18n settings
        i18n = {
            defaultLocale = "en_US",
            localeStorage = "cookie"
        };
        
        // Flash scope settings
        flash = {
            scope = "session",
            properties = {},
            inflateToRC = true,
            inflateToPRC = false,
            autoPurge = true,
            autoSave = true
        };
    }
    
    // Development environment settings
    function development() {
        coldbox.debugMode = true;
        coldbox.handlersIndexAutoReload = true;
        coldbox.handlerCaching = false;
        coldbox.eventCaching = false;
        coldbox.viewCaching = false;
        coldbox.reinitPassword = "";
    }
    
    // Production environment settings
    function production() {
        coldbox.debugMode = false;
        coldbox.handlersIndexAutoReload = false;
        coldbox.handlerCaching = true;
        coldbox.eventCaching = true;
        coldbox.viewCaching = true;
    }
}
```

## Your First Handler

### Step 5: Create a Handler (Controller)

Create `handlers/Blog.cfc`:

```javascript
component extends="coldbox.system.EventHandler" {
    
    // Default action
    function index(event, rc, prc) {
        prc.welcomeMessage = "Welcome to the Blog!";
        prc.posts = [
            {title: "First Post", content: "This is my first blog post", date: now()},
            {title: "Second Post", content: "This is my second blog post", date: now()}
        ];
        
        // Set view to render
        event.setView("blog/index");
    }
    
    // Show individual post
    function show(event, rc, prc) {
        // Get post ID from URL
        prc.postId = rc.id ?: 1;
        prc.post = {
            title: "Sample Post #" & prc.postId,
            content: "This is the content for post #" & prc.postId,
            date: now()
        };
        
        event.setView("blog/show");
    }
    
    // Create new post form
    function create(event, rc, prc) {
        event.setView("blog/create");
    }
    
    // Save new post
    function save(event, rc, prc) {
        // Validate input
        if (!len(trim(rc.title ?: ""))) {
            flash.put("error", "Title is required");
            setNextEvent("blog.create");
        }
        
        // Save logic would go here
        flash.put("success", "Post created successfully!");
        setNextEvent("blog.index");
    }
    
    // Error handler
    function onError(event, rc, prc, faultAction, exception, eventArguments) {
        prc.exception = arguments.exception;
        event.setView("blog/error");
    }
}
```

## Views and Layouts

### Step 6: Create Views

Create `views/blog/index.cfm`:

```html
<cfoutput>
<div class="container">
    <h1>Blog Posts</h1>
    
    <div class="alert alert-info">
        #prc.welcomeMessage#
    </div>
    
    <div class="row">
        <div class="col-md-8">
            <cfloop array="#prc.posts#" index="post">
                <div class="card mb-3">
                    <div class="card-body">
                        <h5 class="card-title">#post.title#</h5>
                        <p class="card-text">#post.content#</p>
                        <small class="text-muted">Posted on #dateFormat(post.date, "mm/dd/yyyy")#</small>
                    </div>
                </div>
            </cfloop>
        </div>
        
        <div class="col-md-4">
            <div class="card">
                <div class="card-header">
                    <h5>Actions</h5>
                </div>
                <div class="card-body">
                    <a href="#event.buildLink('blog.create')#" class="btn btn-primary">Create New Post</a>
                </div>
            </div>
        </div>
    </div>
</div>
</cfoutput>
```

Create `views/blog/show.cfm`:

```html
<cfoutput>
<div class="container">
    <div class="row">
        <div class="col-md-8">
            <article>
                <h1>#prc.post.title#</h1>
                <small class="text-muted">Posted on #dateFormat(prc.post.date, "mm/dd/yyyy")#</small>
                <hr>
                <p>#prc.post.content#</p>
            </article>
            
            <a href="#event.buildLink('blog.index')#" class="btn btn-secondary">← Back to Blog</a>
        </div>
    </div>
</div>
</cfoutput>
```

Create `views/blog/create.cfm`:

```html
<cfoutput>
<div class="container">
    <h1>Create New Post</h1>
    
    <!--- Display flash messages --->
    <cfif flash.exists("error")>
        <div class="alert alert-danger">#flash.get("error")#</div>
    </cfif>
    
    <form action="#event.buildLink('blog.save')#" method="post">
        <div class="form-group">
            <label for="title">Title</label>
            <input type="text" class="form-control" id="title" name="title" required>
        </div>
        
        <div class="form-group">
            <label for="content">Content</label>
            <textarea class="form-control" id="content" name="content" rows="10" required></textarea>
        </div>
        
        <button type="submit" class="btn btn-primary">Save Post</button>
        <a href="#event.buildLink('blog.index')#" class="btn btn-secondary">Cancel</a>
    </form>
</div>
</cfoutput>
```

### Step 7: Update Layout

Edit `layouts/Main.cfm`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>MyApp - ColdBox Application</title>
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <style>
        body { padding-top: 20px; }
        .navbar-brand { font-weight: bold; }
    </style>
</head>
<body>
    <cfoutput>
    <!-- Navigation -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="#event.buildLink('')#">MyApp</a>
            
            <div class="navbar-nav">
                <a class="nav-link" href="#event.buildLink('main.index')#">Home</a>
                <a class="nav-link" href="#event.buildLink('blog.index')#">Blog</a>
            </div>
        </div>
    </nav>
    
    <!-- Flash Messages -->
    <div class="container mt-3">
        <cfif flash.exists("success")>
            <div class="alert alert-success alert-dismissible fade show">
                #flash.get("success")#
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        </cfif>
        
        <cfif flash.exists("error")>
            <div class="alert alert-danger alert-dismissible fade show">
                #flash.get("error")#
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        </cfif>
    </div>
    
    <!-- Main Content -->
    <main class="container mt-4">
        #renderView()#
    </main>
    
    <!-- Footer -->
    <footer class="bg-light mt-5 py-4">
        <div class="container text-center">
            <p>&copy; #year(now())# MyApp. Built with ColdBox Framework.</p>
        </div>
    </footer>
    </cfoutput>
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

## Models and Services

### Step 8: Create a Model

Create `models/BlogService.cfc`:

```javascript
component singleton {
    
    // Properties
    property name="log" inject="logbox:logger:{this}";
    
    // Constructor
    function init() {
        return this;
    }
    
    // Get all posts
    function getAllPosts() {
        // In a real app, this would query a database
        return [
            {
                id: 1,
                title: "Getting Started with ColdBox",
                content: "ColdBox is an amazing CFML framework...",
                author: "John Doe",
                dateCreated: now(),
                published: true
            },
            {
                id: 2,
                title: "Advanced ColdBox Features",
                content: "Let's explore some advanced features...",
                author: "Jane Smith",
                dateCreated: now(),
                published: true
            }
        ];
    }
    
    // Get post by ID
    function getPost(required numeric id) {
        var posts = getAllPosts();
        
        for (var post in posts) {
            if (post.id == arguments.id) {
                return post;
            }
        }
        
        throw(type="PostNotFound", message="Post with ID #arguments.id# not found");
    }
    
    // Create new post
    function createPost(required struct data) {
        // Validate required fields
        if (!structKeyExists(arguments.data, "title") || !len(trim(arguments.data.title))) {
            throw(type="ValidationError", message="Title is required");
        }
        
        if (!structKeyExists(arguments.data, "content") || !len(trim(arguments.data.content))) {
            throw(type="ValidationError", message="Content is required");
        }
        
        // In a real app, save to database
        var newPost = {
            id: randRange(100, 999),
            title: arguments.data.title,
            content: arguments.data.content,
            author: arguments.data.author ?: "Anonymous",
            dateCreated: now(),
            published: arguments.data.published ?: false
        };
        
        log.info("Created new post: #newPost.title#");
        
        return newPost;
    }
    
    // Update existing post
    function updatePost(required numeric id, required struct data) {
        var existingPost = getPost(arguments.id);
        
        // Update fields
        if (structKeyExists(arguments.data, "title")) {
            existingPost.title = arguments.data.title;
        }
        
        if (structKeyExists(arguments.data, "content")) {
            existingPost.content = arguments.data.content;
        }
        
        if (structKeyExists(arguments.data, "published")) {
            existingPost.published = arguments.data.published;
        }
        
        existingPost.dateModified = now();
        
        log.info("Updated post: #existingPost.title#");
        
        return existingPost;
    }
    
    // Delete post
    function deletePost(required numeric id) {
        var post = getPost(arguments.id);
        
        // In a real app, delete from database
        log.info("Deleted post: #post.title#");
        
        return true;
    }
}
```

### Step 9: Update Handler to Use Model

Update `handlers/Blog.cfc`:

```javascript
component extends="coldbox.system.EventHandler" {
    
    // Dependency injection
    property name="blogService" inject="BlogService";
    
    // Default action
    function index(event, rc, prc) {
        try {
            prc.posts = blogService.getAllPosts();
            prc.welcomeMessage = "Welcome to our Blog! We have #arrayLen(prc.posts)# posts.";
        } catch (any e) {
            log.error("Error loading posts: #e.message#", e);
            prc.posts = [];
            prc.welcomeMessage = "Welcome to our Blog!";
            flash.put("error", "Unable to load posts at this time.");
        }
        
        event.setView("blog/index");
    }
    
    // Show individual post
    function show(event, rc, prc) {
        try {
            var postId = val(rc.id ?: 0);
            if (postId == 0) {
                throw(type="InvalidInput", message="Invalid post ID");
            }
            
            prc.post = blogService.getPost(postId);
        } catch (PostNotFound e) {
            flash.put("error", "Post not found");
            setNextEvent("blog.index");
        } catch (any e) {
            log.error("Error loading post: #e.message#", e);
            flash.put("error", "Unable to load post");
            setNextEvent("blog.index");
        }
        
        event.setView("blog/show");
    }
    
    // Create new post form
    function create(event, rc, prc) {
        event.setView("blog/create");
    }
    
    // Save new post
    function save(event, rc, prc) {
        try {
            var postData = {
                title: rc.title ?: "",
                content: rc.content ?: "",
                author: rc.author ?: "Anonymous",
                published: rc.published ?: false
            };
            
            var newPost = blogService.createPost(postData);
            
            flash.put("success", "Post '#newPost.title#' created successfully!");
            setNextEvent("blog.show", {id: newPost.id});
            
        } catch (ValidationError e) {
            flash.put("error", e.message);
            setNextEvent("blog.create");
        } catch (any e) {
            log.error("Error creating post: #e.message#", e);
            flash.put("error", "Unable to create post");
            setNextEvent("blog.create");
        }
    }
}
```

## Database Integration

### Step 10: Configure Database

Create `config/datasources.cfc`:

```javascript
component {
    
    function configure() {
        
        datasources = {
            // Default datasource
            "myapp" = {
                driver = "MySQL",
                host = "localhost",
                port = 3306,
                database = "myapp",
                username = "root",
                password = "",
                options = {
                    useUnicode = true,
                    characterEncoding = "UTF-8"
                }
            }
        };
        
        // Set default datasource
        this.datasource = "myapp";
    }
}
```

### Step 11: Create Database Model

Create `models/Post.cfc`:

```javascript
component accessors="true" {
    
    // Properties
    property name="id" type="numeric";
    property name="title" type="string";
    property name="content" type="string";
    property name="author" type="string";
    property name="dateCreated" type="date";
    property name="dateModified" type="date";
    property name="published" type="boolean";
    
    // Constructor
    function init() {
        setDateCreated(now());
        setPublished(false);
        return this;
    }
    
    // Validation
    function isValid() {
        var errors = [];
        
        if (!len(trim(getTitle()))) {
            arrayAppend(errors, "Title is required");
        }
        
        if (!len(trim(getContent()))) {
            arrayAppend(errors, "Content is required");
        }
        
        return arrayLen(errors) == 0;
    }
    
    // Get validation errors
    function getValidationErrors() {
        var errors = [];
        
        if (!len(trim(getTitle()))) {
            arrayAppend(errors, "Title is required");
        }
        
        if (!len(trim(getContent()))) {
            arrayAppend(errors, "Content is required");
        }
        
        return errors;
    }
}
```

### Step 12: Create Database Service

Create `models/PostDAO.cfc`:

```javascript
component singleton {
    
    // Properties
    property name="dsn" inject="coldbox:setting:datasource";
    property name="log" inject="logbox:logger:{this}";
    
    // Get all posts
    function getAll() {
        var sql = "
            SELECT id, title, content, author, dateCreated, dateModified, published
            FROM posts
            WHERE published = 1
            ORDER BY dateCreated DESC
        ";
        
        var query = queryExecute(sql, {}, {datasource: dsn});
        
        return queryToArray(query);
    }
    
    // Get post by ID
    function getById(required numeric id) {
        var sql = "
            SELECT id, title, content, author, dateCreated, dateModified, published
            FROM posts
            WHERE id = :id
        ";
        
        var params = {id: {value: arguments.id, cfsqltype: "cf_sql_integer"}};
        var query = queryExecute(sql, params, {datasource: dsn});
        
        if (query.recordCount == 0) {
            throw(type="PostNotFound", message="Post with ID #arguments.id# not found");
        }
        
        return queryRowToStruct(query, 1);
    }
    
    // Create new post
    function create(required Post post) {
        var sql = "
            INSERT INTO posts (title, content, author, dateCreated, published)
            VALUES (:title, :content, :author, :dateCreated, :published)
        ";
        
        var params = {
            title: {value: post.getTitle(), cfsqltype: "cf_sql_varchar"},
            content: {value: post.getContent(), cfsqltype: "cf_sql_longvarchar"},
            author: {value: post.getAuthor(), cfsqltype: "cf_sql_varchar"},
            dateCreated: {value: post.getDateCreated(), cfsqltype: "cf_sql_timestamp"},
            published: {value: post.getPublished(), cfsqltype: "cf_sql_bit"}
        };
        
        var result = queryExecute(sql, params, {datasource: dsn, result: "result"});
        
        // Set the generated ID
        post.setId(result.generatedKey);
        
        log.info("Created post: #post.getTitle()# (ID: #post.getId()#)");
        
        return post;
    }
    
    // Update existing post
    function update(required Post post) {
        var sql = "
            UPDATE posts
            SET title = :title,
                content = :content,
                author = :author,
                dateModified = :dateModified,
                published = :published
            WHERE id = :id
        ";
        
        var params = {
            id: {value: post.getId(), cfsqltype: "cf_sql_integer"},
            title: {value: post.getTitle(), cfsqltype: "cf_sql_varchar"},
            content: {value: post.getContent(), cfsqltype: "cf_sql_longvarchar"},
            author: {value: post.getAuthor(), cfsqltype: "cf_sql_varchar"},
            dateModified: {value: now(), cfsqltype: "cf_sql_timestamp"},
            published: {value: post.getPublished(), cfsqltype: "cf_sql_bit"}
        };
        
        queryExecute(sql, params, {datasource: dsn});
        
        log.info("Updated post: #post.getTitle()# (ID: #post.getId()#)");
        
        return post;
    }
    
    // Delete post
    function delete(required numeric id) {
        var sql = "DELETE FROM posts WHERE id = :id";
        var params = {id: {value: arguments.id, cfsqltype: "cf_sql_integer"}};
        
        var result = queryExecute(sql, params, {datasource: dsn, result: "result"});
        
        if (result.recordCount == 0) {
            throw(type="PostNotFound", message="Post with ID #arguments.id# not found");
        }
        
        log.info("Deleted post with ID: #arguments.id#");
        
        return true;
    }
    
    // Helper function to convert query row to struct
    private function queryRowToStruct(required query qry, required numeric row) {
        var result = {};
        var columns = listToArray(qry.columnList);
        
        for (var col in columns) {
            result[col] = qry[col][row];
        }
        
        return result;
    }
    
    // Helper function to convert query to array of structs
    private function queryToArray(required query qry) {
        var result = [];
        
        for (var i = 1; i <= qry.recordCount; i++) {
            arrayAppend(result, queryRowToStruct(qry, i));
        }
        
        return result;
    }
}
```

## Routing

### Step 13: Configure Routes

Edit `config/Router.cfc`:

```javascript
component {
    
    function configure() {
        
        // Set full rewrites
        setFullRewrites(true);
        
        // Set unique URLs
        setUniqueURLs(false);
        
        // Base URL
        if (len(getSetting("AppMapping"))) {
            setBaseURL("http://#cgi.HTTP_HOST##getSetting('AppMapping')#/index.cfm");
        } else {
            setBaseURL("http://#cgi.HTTP_HOST#/index.cfm");
        }
        
        // Your Application Routes
        
        // Blog routes
        route("/blog")
            .withHandler("blog")
            .withAction("index")
            .withName("blog");
            
        route("/blog/create")
            .withHandler("blog")
            .withAction("create")
            .withName("blog.create");
            
        route("/blog/save")
            .withHandler("blog")
            .withAction("save")
            .withName("blog.save")
            .withVerbs("POST");
            
        route("/blog/:id")
            .withHandler("blog")
            .withAction("show")
            .withName("blog.show")
            .where({id: "[0-9]+"});
            
        route("/blog/:id/edit")
            .withHandler("blog")
            .withAction("edit")
            .withName("blog.edit")
            .where({id: "[0-9]+"});
            
        route("/blog/:id/update")
            .withHandler("blog")
            .withAction("update")
            .withName("blog.update")
            .withVerbs("POST,PUT")
            .where({id: "[0-9]+"});
            
        route("/blog/:id/delete")
            .withHandler("blog")
            .withAction("delete")
            .withName("blog.delete")
            .withVerbs("POST,DELETE")
            .where({id: "[0-9]+"});
        
        // API routes
        route("/api/posts")
            .withHandler("api.posts")
            .withAction("index")
            .withName("api.posts")
            .withVerbs("GET");
            
        route("/api/posts")
            .withHandler("api.posts")
            .withAction("create")
            .withName("api.posts.create")
            .withVerbs("POST");
            
        route("/api/posts/:id")
            .withHandler("api.posts")
            .withAction("show")
            .withName("api.posts.show")
            .withVerbs("GET")
            .where({id: "[0-9]+"});
        
        // Default route
        route("/:handler/:action?")
            .withName("generic");
    }
}
```

## Testing

### Step 14: Create Tests

Create `tests/specs/unit/BlogServiceTest.cfc`:

```javascript
component extends="testbox.system.BaseSpec" {
    
    function beforeAll() {
        // Setup
        blogService = createMock("models.BlogService");
    }
    
    function run() {
        
        describe("BlogService", function() {
            
            it("should return all posts", function() {
                var posts = blogService.getAllPosts();
                
                expect(posts).toBeArray();
                expect(posts).toHaveLength(2);
                expect(posts[1]).toHaveKey("title");
                expect(posts[1]).toHaveKey("content");
            });
            
            it("should return a specific post by ID", function() {
                var post = blogService.getPost(1);
                
                expect(post).toBeStruct();
                expect(post.id).toBe(1);
                expect(post.title).toBeString();
            });
            
            it("should throw error for non-existent post", function() {
                expect(function() {
                    blogService.getPost(999);
                }).toThrow("PostNotFound");
            });
            
            it("should create a new post", function() {
                var postData = {
                    title: "Test Post",
                    content: "Test content",
                    author: "Test Author"
                };
                
                var newPost = blogService.createPost(postData);
                
                expect(newPost).toBeStruct();
                expect(newPost.title).toBe("Test Post");
                expect(newPost.id).toBeNumeric();
            });
            
            it("should validate required fields when creating post", function() {
                expect(function() {
                    blogService.createPost({});
                }).toThrow("ValidationError");
                
                expect(function() {
                    blogService.createPost({title: ""});
                }).toThrow("ValidationError");
            });
        });
    }
}
```

Create `tests/specs/integration/BlogHandlerTest.cfc`:

```javascript
component extends="coldbox.system.testing.BaseTestCase" {
    
    function beforeAll() {
        super.beforeAll();
        // Setup application
        setup();
    }
    
    function run() {
        
        describe("Blog Handler Integration Tests", function() {
            
            beforeEach(function() {
                // Reset for each test
                setup();
            });
            
            it("should display blog index page", function() {
                var event = execute(event="blog.index", renderResults=true);
                
                expect(event.getRenderedContent()).toInclude("Blog Posts");
                expect(event.getPrivateValue("posts")).toBeArray();
            });
            
            it("should display individual blog post", function() {
                var event = execute(event="blog.show", eventArguments={id=1}, renderResults=true);
                
                expect(event.getRenderedContent()).toInclude("Getting Started with ColdBox");
                expect(event.getPrivateValue("post")).toBeStruct();
            });
            
            it("should redirect for invalid post ID", function() {
                var event = execute(event="blog.show", eventArguments={id=999});
                
                expect(event.getValue("relocate_event")).toBe("blog.index");
            });
            
            it("should display create post form", function() {
                var event = execute(event="blog.create", renderResults=true);
                
                expect(event.getRenderedContent()).toInclude("Create New Post");
                expect(event.getRenderedContent()).toInclude("<form");
            });
        });
    }
}
```

### Step 15: Run Tests

```bash
# Install TestBox
box install testbox

# Run all tests
box testbox run

# Run specific test suite
box testbox run directory=tests/specs/unit

# Run with coverage
box testbox run coverage=true
```

## Advanced Features

### Step 16: Interceptors

Create `interceptors/SecurityInterceptor.cfc`:

```javascript
component extends="coldbox.system.Interceptor" {
    
    function configure() {
        // Configuration
    }
    
    function preProcess(event, interceptData) {
        var rc = event.getCollection();
        var prc = event.getPrivateCollection();
        
        // Security checks
        if (event.getCurrentEvent() contains "admin" && !isUserLoggedIn()) {
            flash.put("error", "Please log in to access admin area");
            setNextEvent("login");
        }
        
        // Log all requests
        log.info("Processing event: #event.getCurrentEvent()#");
    }
    
    function postProcess(event, interceptData) {
        // Post-processing logic
        var renderData = event.getRenderData();
        
        if (structKeyExists(renderData, "contentType") && renderData.contentType == "application/json") {
            // Add security headers for API responses
            event.setHTTPHeader(name="X-Content-Type-Options", value="nosniff");
            event.setHTTPHeader(name="X-Frame-Options", value="DENY");
        }
    }
    
    private function isUserLoggedIn() {
        return structKeyExists(session, "user") && structKeyExists(session.user, "id");
    }
}
```

### Step 17: Modules

Create `modules_app/api/ModuleConfig.cfc`:

```javascript
component {
    
    // Module properties
    this.title = "API Module";
    this.author = "Your Name";
    this.webURL = "http://www.yoursite.com";
    this.description = "RESTful API for the application";
    this.version = "1.0.0";
    
    // Model namespace
    this.modelNamespace = "api";
    
    // CF Mapping
    this.cfmapping = "api";
    
    function configure() {
        
        // Module settings
        settings = {
            apiVersion = "v1",
            defaultFormat = "json"
        };
        
        // Layout settings
        layoutSettings = {
            defaultLayout = ""
        };
        
        // Interceptors
        interceptors = [
            {class="#moduleMapping#.interceptors.APIInterceptor"}
        ];
    }
    
    function onLoad() {
        // Module loading
        log.info("API Module loaded successfully");
    }
    
    function onUnload() {
        // Module unloading
        log.info("API Module unloaded");
    }
}
```

### Step 18: Caching

Update `config/ColdBox.cfc` to add caching:

```javascript
// Add to configure() function
cache = {
    providers = {
        default = {
            objectDefaultTimeout = 60,
            objectDefaultLastAccessTimeout = 30,
            useLastAccessTimeouts = true,
            reapFrequency = 2,
            freeMemoryPercentageThreshold = 0,
            evictionPolicy = "LRU",
            evictCount = 1,
            maxObjects = 300,
            objectStore = "ConcurrentStore"
        },
        template = {
            objectDefaultTimeout = 120,
            objectDefaultLastAccessTimeout = 30,
            useLastAccessTimeouts = true,
            reapFrequency = 2,
            freeMemoryPercentageThreshold = 0,
            evictionPolicy = "LRU",
            evictCount = 2,
            maxObjects = 50,
            objectStore = "ConcurrentStore"
        }
    }
};
```

Use caching in your handlers:

```javascript
// In BlogService.cfc
function getAllPosts() {
    return cache.getOrSet("blog.allPosts", function() {
        // Expensive database operation
        return postDAO.getAll();
    }, 30); // Cache for 30 minutes
}
```

## Deployment

### Step 19: Production Configuration

Create `config/environments/production.cfc`:

```javascript
component {
    
    function configure() {
        
        // Production settings
        coldbox.debugMode = false;
        coldbox.handlersIndexAutoReload = false;
        coldbox.handlerCaching = true;
        coldbox.eventCaching = true;
        coldbox.viewCaching = true;
        coldbox.reinitPassword = "your-secure-password";
        
        // Logging for production
        logBox.appenders.files = {
            class = "coldbox.system.logging.appenders.RollingFileAppender",
            properties = {
                filename = "logs/coldbox",
                autoExpand = true,
                fileMaxSize = 2000,
                fileMaxArchives = 5
            }
        };
        
        logBox.root.levelMin = "WARN";
        logBox.root.appenders = "files";
    }
}
```

### Step 20: Build and Deploy

```bash
# Create production build
box coldbox create app-template production

# Or use CommandBox for deployment
box server start production=true

# Deploy to server (example with rsync)
rsync -av --exclude='.git' --exclude='tests' ./ user@server:/path/to/app/
```

## Summary

You now have a complete ColdBox application with:

✅ **MVC Architecture**: Handlers (Controllers), Views, and Models
✅ **Dependency Injection**: Using WireBox for service management
✅ **Database Integration**: DAO pattern with query execution
✅ **Routing**: RESTful URL routing
✅ **Testing**: Unit and integration tests with TestBox
✅ **Caching**: Built-in caching mechanisms
✅ **Security**: Interceptors for security and logging
✅ **Modules**: Modular application architecture
✅ **Production Ready**: Environment-specific configurations

## Next Steps

1. **Learn More Advanced Features**:
   - WireBox advanced DI patterns
   - CacheBox distributed caching
   - LogBox advanced logging
   - Async programming

2. **Explore ColdBox Modules**:
   - cbSecurity for authentication/authorization
   - cbValidation for form validation
   - cbMailServices for email
   - cbElasticsearch for search

3. **Performance Optimization**:
   - Query optimization
   - Caching strategies
   - CDN integration
   - Load balancing

4. **DevOps Integration**:
   - CI/CD pipelines
   - Docker containerization
   - Monitoring and logging
   - Automated testing

This guide provides a solid foundation for building enterprise-level CFML applications with ColdBox!

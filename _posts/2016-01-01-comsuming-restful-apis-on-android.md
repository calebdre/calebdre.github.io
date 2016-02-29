---
layout: post
title: Comsuming RESTful APIs on Android
---

So you just created or are given a RESTful API and you need a way to query it to give or receive informationfrom it in 
your Android application. In this article, we'll create an easy and scalable solution that you can put into your project
and build off of. Let's get into it!  
  
Doing a quick [google search](https://www.google.com/search?q=http+request+android&oq=http+request+android&aqs=chrome..69i57l2j69i60l4.2250j0j1&sourceid=chrome&es_sm=91&ie=UTF-8)
you'll get articles telling you to use the `HttpClient` and `HttpResponse` but if you copy and paste the code into 
Android Studio, it doesn't recognize those classes. This is how make reuqests using the Apache Http library, which
the Android framework and since dropped support for. So that's a no go.  
  
Doing HTTP requests in plain Java won't be fun at all, so let's check out some libraries the can make things easier. 
The most popular HTTP client libraries specific to Android are [Retrofit](http://square.github.io/retrofit/), 
[OkHttp](http://square.github.io/okhttp/), and [Volley](http://developer.android.com/training/volley/request.html).
After reading some reviews, comparisons, and examples, OkHttp seems to be the simplest and most straightforward to use. 
I especially like it because the code is easy to read and to figure out what's going on.  
  
So now that we know which library we're going to use, let's go over the use cases that want to be able to have an easy solution to given [RESTful conventions](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#method-override):  

Let's think about how we want to make HTTP requests in our app. Suppose we have a news app and we want to display all the news for today:
{% highlight java %}
public class NewsActivity{
	
	@Override
	protected void onCreate(@Nullable Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_news);
		// ...
		// go get the news
		// ... 
	}
}
{% endhighlight %}  
  
Putting all that network code in the activity isn't a good idea - it violates the [single responsibility principle LINK](LINK HERE). The activity
should be in charge of manipulating the view, so we will extract all the HTTP logic to another class. Here would be an ideal implementation (edited from above):  
  
{% highlight java %}
public class NewsActivity{
	//...	
	NewsService newsService = new NewsService();
	@Override
	protected void onCreate(@Nullable Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_news);
		// ...
		// go get the news
		newsService.getNews(new GetNewsCallback(){
			public void onNewsReceived(List<NewsArticle> newsItems){
				// attach the news items to a list
			}	
		}); 
		// ... 
	}
}
{% endhighlight %}  

Looks like a good start for a nice and easy implementation. Let's create a directory anywhere in the projects 
called `RestServices` and put a `NewsService` class:

{% highlight java %}
public class NewsService{
	public void getNews(GetNewsCallback callback){
		// network logic to receive news
		// put news in a List
		// give the list to the consumer's callback
		callback.onNewsReceived(news); 
	}
	
	public interface GetNewsCallback{
		void onNewsReceived(List<NewsArticle> newsItems);	
	}
}
{% endhighlight %}  

We create the `getNews` method that we used in the implementation in the activity that accepts an interface that we create. 
When the consumer of the method calls it, they have to implement that interface which results in the code that we saw in 
our desired implementation. We can then call `callback.onNewsReceived()` and pass it the data that we received from the api. 
Calling that function will execute the stuff that we put inside the anonymous class in the implementation.  
  
Now we need to figure out how to make the network request. Looking at the [OkHttp documentation LINK](LINK HERE),
this is the basic structure of a request: 
{% highlight java %}
// instantiate a builder that builds the request
Builder builder = new Builder();

// add the various elements of the request
builder.url("your url");
builder.get();

// build the request
Request okRequest = builder.build();

// pass it to a client and dispatch it to execute in another thread
// also, give it an anonymous class so that you can do something with the response
client.newCall(okRequest).enqueue(new Callback() { 
	
	// method that's called when the request doesn't go through
	@Override
	public void onFailure(Request request, IOException e) {}

	// method that's called when the request succeeds
	@Override
	public void onResponse(Response response) throws IOException {
		// do stuff with the json
		String json = response.body().string());
	}
});
{% endhighlight %}  
  
Let's incorporate this into the `getNews` method:  
  
{% highlight java %}
public void getNews(GetNewsCallback callback){
	// network logic to receive news
	Builder builder = new Builder();

	builder.url("http://newsnetwork.com/api/news"); // extremely fictitious api
	builder.get();

	Request okRequest = builder.build();
	
	client.newCall(okRequest).enqueue(new Callback() { 
		@Override
		public void onFailure(Request request, IOException e) {}

		@Override
		public void onResponse(Response response) throws IOException {
			String json = response.body().string());
			
			// put news items in a List
			// give the list to the consumer's callback
			callback.onNewsReceived(news);
		}
	}); 
}
{% endhighlight %}  
  
Now that we've got the network call, we need to figure out how to do something with the response json.
A quick google of json parsing libraries yields [Gson lINK](LINK) and [Jackson LINK](lINK) as the most popular.
After reading some reviews, comparisons, and examples, Gson seems like the better choice due to great documentation
and a super straightforward implementation. It works by using reflection to match keys and their values in a json string to
fields in a [POJO LINK](LINK HERE).  
  
Let's create our news item POJO:  
{% highlight java %}
public class NewsArticle{
	private String title, content, date, url, thumbnailImage;
	private Author author;
	private List<Comment> comments;
	
	// -- getters --
}
{% endhighlight %}  
  
This class represents a news article entity. Whenever we need to do something with a new article,
we can use this object to store all the information about, manipulate, and display the entity.  
    
Now let's edit the `getNews` method using Gson:    
{% highlight java %}
public void getNews(GetNewsCallback callback){
	// network logic to receive news
	Builder builder = new Builder();

	builder.url("http://newsnetwork.com/api/news"); // extremely fictitious api
	builder.get();

	Request okRequest = builder.build();
	
	client.newCall(okRequest).enqueue(new Callback() { 
		@Override
		public void onFailure(Request request, IOException e) {}

		@Override
		public void onResponse(Response response) throws IOException {
			String json = response.body().string());
			
			// transform the json string to an object that Gson accepts
			JsonElement gsonJson = new JsonParser().parse(json);
			
			List<NewsArticle> articles = new Gson().fromJson(gsonJson, new TypeToken<List<NewsArticle>>(){}.getType());
			
			callback.onNewsReceived(articles);
		}
	}); 
}
{% endhighlight %}  
  
Gson's `fromJson` method accepts a `JsonElement` object and a `Type` object (from java.reflection.) You could easily 
call it like so:  
    
{% highlight java %}
NewsArticle article = new Gson().fromJson(jsonElementObj, NewsArticle.class);
{% endhighlight %}  

However when you have a list of elements in the json, you can't give it `List<NewsArticle>.class` as an argument,
so we give it a `TypeToken`. We transform a json array of objects into a list of objects with Gson like so:  

{% highlight java %}
Type listType = new TypeToken<List<yourobject>>(){}.getType();
List<yourobject> yourobjects = new Gson().fromJson(jsonElementObj, listType);
{% endhighlight %}  
  
That's what we did inside the `getNews` method. Once we got the list, we passed it to the `onNewsReceived` method 
on the callback.
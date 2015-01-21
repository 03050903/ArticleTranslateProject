---
layout: post
title: "Picasso����RequestHandler��API"
date: 2015-01-21 17��11
comments: true
categories: Picasso RequestHandler API
---

#Picasso����RequestHandler��API

�����ҷ�����һƪʹ��Picasso�ⴴ���Զ������ص����£�������ƪ���º��ұ���֪����һ���汾��Picasso��Picasso2.4������������һ���µ�API��

����µ�API����RequestHandler API������[Lucas Rocha](http://lucasr.org/)��һ��[PR](https://github.com/square/picasso/pull/512).

##����

��ʹ����ʹ���Զ��������ܹ�������������µ����⣬�µ�API�ܽ������������һЩ���ƣ���

1. DowloaderӦ�ý������������أ�������Ǵ�����ͼƬ��downloader�ͷŴ��˵ط���

2. ������Ǽ��غܶ�ط�ȡ�õ�λͼ��ʹ��Dowloaders���Եù��ڸ��ӡ�

3. ֻ��һ��Picassoʵ���������κ����󣬱ȴ���һ��Piccasso ʵ������ĳ������µ����飬��Ĭ��ʵ���������ಿ��Ҫ��һЩ��

##�������
RequestHandler��һ��Picasso��ʹ���߿���ͨ�����̳�����չ���Զ���ͼ������߼����еĹ����ࡣ

�����ڿ�ʼ����Picassoʵ������Picasso������RequestHandler��list���佫����ѯ�������Ƿ���Դ�������

1. ���б��еĵ�һ��λ�����Ƿ�����һ��Ĭ�ϵ�Picasso ��RequestHandler������������Android������Դ���Ժ��һ����Ϊʲô������б��еĵ�һ��RequestHandler�ģ� ��
[ResourceRequestHandler](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/ResourceRequestHandler.java)

2. ���б��е�λ�ú����ǿ��Է��֣��Զ���RequestHandler�Ѿ����뵽���ǵ�Picassoʵ����ע�⣬ֻ�����Ǵ����Զ����Picassoʵ�������ǿ��Բ�����Զ����RequestHandler��
```java
//Create Picasso instance with custom RequestHandler's
Picasso picassoInstance = = new Picasso.Builder(mContext.getApplicationContext())
        .addRequestHandler(new MyFirstRequestHandler())
        .addRequestHandler(new MySecondquestHandler())
        //..
        .addRequestHandler(new MyLastRequestHandler())
        .build();
```
3. ����б���ʣ��Ĳ��ֽ���ٳ�������Ĭ�ϵ�Picasso RequestHandler

- [MediaStoreRequestHandler](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/MediaStoreRequestHandler.java)
- [ContentStreamRequestHandler](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/ContentStreamRequestHandler.java)
- [AssetRequestHandler](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/AssetRequestHandler.java)
- [FileRequestHandler](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/FileRequestHandler.java)
- [NetworkRequestHandler](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/NetworkRequestHandler.java)

������ǿ��� [Picasso contructor](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/Picasso.java#L144)���ǻᷢ��requestHandler ����ι��ɵġ�
```java
final int extraCount = (extraRequestHandlers != null ? extraRequestHandlers.size() : 0);
final List<RequestHandler> allRequestHandlers = new ArrayList<RequestHandler>(7 + extraCount);
// ResourceRequestHandler needs to be the first in the list to avoid
// forcing other RequestHandlers to perform null checks on request.uri
// to cover the (request.resourceId != 0) case.
allRequestHandlers.add(new ResourceRequestHandler(context));
if (extraRequestHandlers != null) {
  allRequestHandlers.addAll(extraRequestHandlers);
}
allRequestHandlers.add(new ContactsPhotoRequestHandler(context));
allRequestHandlers.add(new MediaStoreRequestHandler(context));
allRequestHandlers.add(new ContentStreamRequestHandler(context));
allRequestHandlers.add(new AssetRequestHandler(context));
allRequestHandlers.add(new FileRequestHandler(context));
allRequestHandlers.add(new NetworkRequestHandler(dispatcher.downloader, stats));
requestHandlers = Collections.unmodifiableList(allRequestHandlers);
```

����Ĵ����е�ע��˵��ΪʲôResourceRequestHandler���б��еĵ�һ����ԭ��

ResourceRequestHandler��Ҫ�������б��У��Ա�����ʹ����RequestHandlers��request.uri����null��飬���ǣ� request.resourceId �� = 0 ���������
Ҫ�����һ�㣬������Ҫ֪����δ���һ���Զ����RequestHandler��

ÿ���Զ����RequestHandler�̳��Գ�����[RequestHandler.java](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/RequestHandler.java)��ͬʱ��ÿ���Զ����RequestHandler����̳����ĳ��󷽷���

- [public abstract boolean canHandleRequest(Request)](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/RequestHandler.java#L94)

-- ���ResquestHandler�ܴ���������������������true�����򷵻�false

- [public abstract Result load(Request)](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso/RequestHandler.java#L102)

-- �������������д���Զ���ĵ���Picasso����ͼƬ���߼�����canHandleRequest���κ�RequestHandler���ã���Picasso�������ǣ���ֻҪ����һ������true ���������׳�IllegalStateException���쳣 ��
```java 
static BitmapHunter forRequest(Picasso picasso, Dispatcher, dispatcher,Cache cache, Stats stats, Action action) {
    Request request = action.getRequest();
    List<RequestHandler> requestHandlers = picasso.getRequestHandlers();
    final int count = requestHandlers.size();
    for (int i = 0; i < count; i++) {
        RequestHandler requestHandler = requestHandlers.get(i);
        if (requestHandler.canHandleRequest(request)) {
            return new BitmapHunter(picasso, dispatcher, cache, stats, action, requestHandler);
        }
    }

    throw new IllegalStateException("Unrecognized type of request: " + request);
}
```

���ResourcesRequestHandler ���б�ĵ�һ����������󣨷���false �� ������֪���� request.uri����Ϊ�ա������ΪʲôResourcesRequestHandler���б��еĵ�һ����

�Զ����RequestHandler����Ҫ��

���ط���һ������RequestHandler�����ʵ��ͬʱ�����ʹӣ����̣��ڴ棬���磩���ص�λͼ��

�ص�ָ�����ǣ�������Ȼ�ܼ̳��Զ����Downloader���Ӷ�����Picassoʵ������������ͼƬ�ķ�ʽ�����������Զ����Downloader������ר������NetworkRequestHandler ���б�����һ��������RequestHandler��ֻ��������������

```java 
private static final String SCHEME_HTTP = "http";
private static final String SCHEME_HTTPS = "https";

//..

@Override public boolean canHandleRequest(Request data) {
String scheme = data.uri.getScheme();
    return (SCHEME_HTTP.equals(scheme) || SCHEME_HTTPS.equals(scheme));
}

//..
```
##����

OK,���ڣ��Ҳ�������֪��...����δ����Զ����RequestHandler��Well���Ǻܺý��͡�

�ҽ���ʹ�ú�һ����DropBox�����������ֽ��͵������ԡ�

����������Ҫ����һ���̳���RequestHandler����

```java 
public class DropBoxRequestHandler extends RequestHandler {

    private final DropBoxInteractor mDropBoxInteractor;

    public DropBoxRequestHandler(DropBoxInteractor dropBoxInteractor) {
        mDropBoxInteractor = dropBoxInteractor;
    }

    @Override
    public boolean canHandleRequest(Request data) {
        return false;
    }

    @Override
    public Result load(Request data) throws IOException {
        return null;
    }
}
```

Ȼ��������Ҫ������canHandleRequest(Request)������ʹ��DropBoxRequestHandler������������Picasso����֪�����Ƿ�Ӧ��ʹ��handler���߱�����ѯ��

```java 
    @Override
    public boolean canHandleRequest(Request data) {
        String scheme = data.uri.getScheme();
        return (SCHEME_DROPBOX.equals(scheme));
    }
```

��Picassoѯ��DropBoxRequestHandler�Ƿ��ܴ���dropbox������ʱ���������Ϊtrue�������Ե�Picasso������ʾbitmapʱ���������������������

```java 
 @Override
    public Result load(Request data) throws IOException {
        InputStream in = mDropBoxInteractor.getThumbnailStream(uri.toString().replace(SCHEME_DROPBOX+":",""));
        return new Result(BitmapFactory.decodeStream(in), NETWORK);
    }
```

���һ��������Ҫ��������������DropBoxRequestHandler��Picasso��ʵ����

```java 
 Picasso picassoInstance = new Picasso.Builder(mContext.getApplicationContext())
            .addRequestHandler(new DropBoxRequestHandler(dropBoxInteractor))
            .build();
When you have created you own picasso instance, make sure you don't use with(Context context). That's a static method that initializes the default Picasso instance. 
You just need to call picassoInstance.load() directly after you have built your instance, like in the following example:

picassoInstance.load(SCHEME_DROPBOX+":"+dropboxItem.getPath()).into(imageView);
```

���ˣ�����Picasso��ʵ���ܹ���Ĭ�ϵ�Picassoʵ��һ��������ͬ�����󣬰���dropbox������ͼ��


##����

RequestHandler���������飺

���ǲ���Ҫʹ�ý���downloader�Ķ���ȥ����bitmap��

�����Զ���Ŀ�����ģ�黯�ķ�ʽ����bitmap������ÿ��RequestHandler�������Լ�����������ɡ�

�Զ����Picassoʵ�������������κη���RequestHandler������������

������Ȼ�ܹ��Զ������ǵ�HTTP��HTTPS URI���ط�����

�����������µ�ʵ���Һܸ��ˣ�����˵���Ǹ�л����Ϊ���������������Ĺ�����

��ϣ������ϲ���������Ҽ���coding!

---


ԭ�ĵ�ַ��[http://blog.jpardogo.com/requesthandler-api-for-picasso-library/](http://blog.jpardogo.com/requesthandler-api-for-picasso-library/)

���ߣ�[jackuhan](https://github.com/jackuhan) У�ԣ�[У����ID](https://github.com/У����ID)

# https_with_volley

If you are not familiar with Android volley read the following articles Android Developers - Volley or AndroidHive - Volley.

If you haven't already created your self-signed certificate do the following steps:

Create self-signed certificate (Windows - Apache 2.2):
Follow the instructions of blog.lifebloodnetworks.com to create your own self-signed certificate and install it on your Apache server. 

Add the certificate to a keystore file

After you have succesfully added your .cer and .key files in your Apache server you have to add the .cer file in a keystore file. To create the keystore file:
Download the latest bcprov-jdk*on-*.jar version of bouncy castle.
Store it in a known place (Desktop for example)
Go to "C:\Program Files\Java\jre7\bin" or anywhere else you have installed your Java. jre7 will be the version of your java jre
Open a command line (shortcut Shift + right click and press open a command line here).

### Enter the following command:
keytool -importcert -v -trustcacerts -file "C:\path to Desktop\your.cer" -alias enter_your_alias -keystore "C:\path to Desktop\keystore.bks" -provider org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath "C:\path to Desktop\bcprov-jdk*on-*.jar" -storetype BKS -storepass your_pass_without_quotes

Add your keystore in your Android appliction

To add your previously created keystore in your Android project go to your project in Eclipse or Android Studio. Go to res and create a raw folder if it is not already exist. Paste your keystore.bks there.
Make Https Volley request using HurlStack

The easiest way to make an https volley request is by using the default Android volley HurlStack.


## Example

This example explains the usage of keystore and how to read the crt. If you want more details on how to use android volley read the links about volley in the beginning of the post.

### 1. Create a singleton VolleyProvider

public class VolleyProvider{
 private static VolleyProvider instance;
 private RequestQueue queue;
 private HurlStack hurlStack;
 
 private VolleyProvider(Context ctx){
  try {
   InputStream instream = ctx.getApplicationContext().getResources()
     .openRawResource(R.raw.keystore);
   KeyStore trustStore = KeyStore.getInstance("BKS");
   try {
    trustStore.load(instream, "yourpassword".toCharArray());
    
   } catch (Exception e) {
    e.printStackTrace();
   } finally {
    try {instream.close();} catch (Exception ignore) {}
   }
   
   String algorithm = TrustManagerFactory.getDefaultAlgorithm();
   TrustManagerFactory tmf = TrustManagerFactory
     .getInstance(algorithm);
   tmf.init(trustStore);
   SSLContext context = SSLContext.getInstance("TLS");
   context.init(null, tmf.getTrustManagers(), null);

   SSLSocketFactory sslFactory = context.getSocketFactory();
   hurlStack = new HurlStack(null, sslFactory);
   queue = Volley.newRequestQueue(ctx, hurlStack);
  } catch (Exception e) {
   e.printStackTrace();
  }
 }
 
 public static VolleyProvider getInstance(Context ctx){
  if(instance == null){
   instance = new VolleyProvider(ctx);
  }
  return instance;
 }
 
 public RequestQueue getQueue(){
  return this.queue;
 }
 
 public <T> Request<T> addRequest(Request<T> req) {
              return getQueue().add(req);
    }
 
 public <T> Request<T> addRequest(Request<T> req, String tag) {
             req.setTag(tag);
  return getQueue().add(req);
    }

}

### 2. Make the request from any other Activity class

     VolleyProvider.getInstance(this).addRequest(myrRequest);

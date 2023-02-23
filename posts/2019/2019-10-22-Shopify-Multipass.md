---
title: Shopify Multipass in .NET Core 3.0
category: Rovani in C♯
tags:
- shopify
- dotnetcore
date: 2019-10-22
---

Shopify has a feature that allows an external service to automatically log a user into the store from a third-party application. This is commonly used when integrating a Shopify store with a larger website product like Wordpress, DNN, or Drupal. Shopify calls this feature "[Multipass](https://help.shopify.com/en/api/reference/plus/multipass)". Of course, this means for the last two days, all I can hear in my head is "Leeloo Dallas Multipass"; but even worse than having it on repeat (in my head) is that _no one else on the project gets the reference_.

[![Leeloo Dallas Auth0 Multipass](/images/leeloo-dallas-auth0.jpg)](https://www.youtube.com/watch?v=8bF5ft-oOWU)

Now that that's out of the way - on to the project at hand. For a client that we are working with at BlueBolt, the desire is to use [Azure AD B2C](https://azure.microsoft.com/en-us/services/active-directory-b2c/) as the backing authentication mechanism. The user experience is simple: the customer clicks the "Login" button, they are sent to an Azure AD B2C login form, then redirected back to the Shopify store. However, as we all know, nothing is as easy as it seems.

The implementation of the actual authentication is being done by a different consulting firm. For our part, we are just working on the front-end of the Shopify store (creative, templates, etc.) and I was brought on to support the external integrations.

After hours of working through how to get a valid Multipass URL to work, I finally was able to put together [the following gist](https://gist.github.com/drovani/df732f165d9735ad635707db1020d55d#file-shopify-multipass-demo-cs):

```csharp
string secret = "[shopify-multipass-secret]";
string store = "[shopify-store]";
var json = System.Text.Json.JsonSerializer.Serialize(new {
	email = "[customer-email]",
	created_at = DateTime.Now.ToString("O"),
	identifier = "[customer-uid]",
	//remote_ip = ""
});

var hash = System.Security.Cryptography.SHA256.Create().ComputeHash(Encoding.UTF8.GetBytes(secret));
var encryptionKey = new ArraySegment<byte>(hash, 0, 16).ToArray();
var signatureKey = new ArraySegment<byte>(hash, 16, 16).ToArray();

var initvector = new byte[16];
new System.Security.Cryptography.RNGCryptoServiceProvider().GetBytes(initvector);

byte[] cipherData = new Func<byte[], byte[], byte[]>((iv, key) =>
{
	byte[] encrypted;
	using (var aes = System.Security.Cryptography.Aes.Create())
	{
		aes.Key = encryptionKey;
		aes.IV = iv;
		aes.Mode = System.Security.Cryptography.CipherMode.CBC;

		var encryptor = aes.CreateEncryptor(aes.Key, aes.IV);
		using (MemoryStream ms = new MemoryStream())
		using (System.Security.Cryptography.CryptoStream cs = new System.Security.Cryptography.CryptoStream(ms, encryptor, System.Security.Cryptography.CryptoStreamMode.Write))
		{
			using (StreamWriter sw = new StreamWriter(cs))
			{
				sw.Write(json);
			}
			encrypted = ms.ToArray();
		}
	}
	return encrypted;
})(initvector, encryptionKey);

byte[] cipher = initvector.Concat(cipherData).ToArray();
byte[] signature = new HMACSHA256(signatureKey).ComputeHash(cipher);
string token = Convert.ToBase64String(cipher.Concat(signature).ToArray()).Replace("+", "-").Replace("/", "_");

string url = $"https://{store}.myshopify.com/account/login/multipass/{token}";
```

## Shopify Provides No Help

![Invalid Multipass Request error](/images/shopify-invalid-multipass-request.png)

The only help that Shopify provides to identify the problem is "Invalid Multipass request" and a `Request ID` at the bottom. However, this is no way to get any details about what the problem is - not even an API call (that I could find) that lets me take the `Request ID` and look at logs on the request. Instead, I had to just keep trying (over and over and over) until I happened to stumble on the right answer — and then it just worked.

## Never Hand-code JSON

I am not really sure what I thought it was the _right idea_ to hand-code the JSON, but that's how I started. Looking back, I don't even know why I thought to switch it to an anonymous object that a JSON library would serialize. However, that was the final task that took a non-functional Multipass request to one that worked.

In fact, when I sent the gist off to the vendor to show them a working example, they called me back and said they couldn't get it working in their code. After doing a screen share, I saw the developer had also hand-coded the JSON. I suggested he change it to the same that I had and low-and-behold, it suddenly started working. My gist was in .NET Core 3.0, which meant they didn't have access to the `System.Text.Json` library. However, using [Newtonsoft's Json.NET](https://www.newtonsoft.com/json), we were able to quickly get a solution that worked.

## Shopify App for Azure AD B2C

Now I'm wondering if there is any kind of a market for a Shopify App that acts as the necessary middle-ware between Shopify's Login page and Microsoft's Azure AD B2C.

If anyone out this is interested in developing it with me, hit me up. It could be an interesting project.

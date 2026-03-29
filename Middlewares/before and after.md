
## Avoid calling next after sending response:
- You can have multiple middleware editing the http response content
- You cannot touch an http response after editing the header or status code
- That's because: http response is stream and it is not closed
- But if you edit a header or status code the http request is done and it should be returned and never edited
- You can change status code **before** writing response but never change it after writing to response (cause `response.WriteAsync()` causes the result to be returned as default 200 you can't change it later)
![[Pasted image 20260305095321.png]]


## Middleware before and after:
- All code before await next() is executed when request enters
- All code after await next() is executed when request leaves
M1 before
	M2 before
		 M3 before
		  M3 after
	 M2 after
 M1 after

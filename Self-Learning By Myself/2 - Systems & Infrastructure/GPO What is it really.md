## **1) 2 Kinds of configuration :

A GPO (Group Object Policy) is used to apply policies to an OU (Object Unit : Computers or Users).

This differentiation happens around only one question : Should this thing be done even if there is no user logged in ? 

If it's yes --> Computer Configuration / Else --> User Configuration.

Normally you have to choose between those branches every time and leave the other empty. And it's more secure to, no matter what, identify simply where a potential failure/error could come from. 
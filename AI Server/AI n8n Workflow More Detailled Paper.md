## **Why I Wanted To Explain More ?**

- After reading again my before paper [AI Server Automated AI-CyberAgent logs analyzer](AI%20Server%20Automated%20AI-CyberAgent%20logs%20analyzer.md) I realized it's confusing for everyone reading it. I want to make a better explanation and go deeper into each node.
- Now you can see the Pre-Final Version. In the future each node will be upgraded. I want to add more logs to the final analyse, to have an "instant" mod for critical log, to upgrade some prompts and vector filters.

![](Pasted%20image%2020260615112121.png)

## **The First Node : Schedule Trigger**

- The easiest one, you just have to chose your interval. In my case : 15 minutes.

## **The Second Node : Http GET + Loki **


 ![](Pasted%20image%2020260615113653.png)

- The GET + URL is a request to the API of Loki to get the data and its range to avoid saturating the server.
- The query and its "level=~" are used to target error levels named Warning/Error or Critical.
- The start and the end module are there for pointing the last 15 minutes to analyze. 
- The limit stands to protect the system from a logs tsunami attack.
- Why the command line is : {{ Math.floor((Date.now() - 15 * 60 * 1000) / 1000) }}000000000
	- This took the current milliseconds count and translate it into seconds by Math.floor and the following operation and the 9 zeros is a syntax for loki that requests a nanoseconds timestamp.

- But there was some problem : 
	- 

## **The Third Node : Http GET + Loki **

- The limit stands to protect the system from a logs tsunami attack.
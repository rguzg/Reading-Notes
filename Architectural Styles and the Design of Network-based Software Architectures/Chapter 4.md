# Designing the Web Architecture: Problems and Insights
https://www.ics.uci.edu/~fielding/pubs/dissertation/web_arch_domain.htm

*Writers notes are in itallics*

This chapter talks about the requirements of the World Wide Web architecture as well as the problems faced when designing improvements to its key communication protocols. The author also hypothesizes about an architectural style that could be used to guide the design of a modern Web architecture. 

## WWW Application Domain Requirements
- Tim Berners-Lee goal for the web was to be a shared information space where people and machines could communicate with eachother. This goal led to the following requirements for the WWW architecture:
    - A way for people to store and structure their own information so that it could be used by themselves and others
    - A way to reference and structure information stored by others so that it would not be neccesary for everyone to keep local copies

*The realization of these requirements can be seen in the HTML language and in URLs*

- The intended users for the World Wide Web were people that used a wide range of computers, operating systems and file formats so finding a way of building a consistent interface that would work on all sorts of computers was an active challenge during the development of the World Wide Web.

### Low Entry-barrier
- As the creation and structuring of information was voluntary, a low entry-barrier was necessary to attract as many people as possible
- Hypermedia was chosen as the user interface for the WWW because hyperlinks could be used to point to other documents, guiding the user through the application

*Hypermedia is a document that can include images, audio, video text and most importantly, hyperlinks or links to other documents*

- People could navigate through information using hyperlinks, but since it's easier to access information within large database through a search interface, the Web also has the ability to perform queries by providing user-entered data to a service and rendering the results as hypermedia

*This past paragraph can be interpreted as a really high abstraction of the concept of sending a request to a server to get information about an specific resource and getting a response that includes an HTML page. It's evidently easier to query a server about a resource than navigating through all the resources contained within the server trying to find the resource you're looking for*

- For authors, a primary requirement was the ability to create content even if the system was not avaialble, or whether they were connected to the Internet or not. 

- Authors were also expected to keep their notes in this format, whether they were connected to the Internet or not. This meant that the system must also allow to access content even if part of the document or references to other documents are unavailable. As a side effect of this, references to resources could be included in documents even if the referenced document didn't exist yet.

- Authors were encouraged to collaborate in the development of information sources, so references needed to be easy to communicate. The author uses the analogy that references had to be able to be written on the back of a napkin.

*Even to this day, this three requirements are still respected on the modern web (kinda). An internet connection is just used to get resources from remote computers. Any webpage or web app can be ran without being connected to the internet (as long as you have all of the resources that the application needs) and if you place a link to a non-exsitent resource, the worst thing that could happen is that you get a 404 error*

- Another requirement was that the hypertext authoring language needed to be simple and capable of being created using existing tools. *Remember that one of the primary goals was to make the process of creating hypermedia easy*

- The last requirement related to the low entry-barrier is that communicaton and interaction between computers had be simple. This was solved by defining all protocols related to the WWW as text; allowing the WWW to be tested by using existing network tools. *No additional processing or protocols was required to connect to other computers, you just connected to another computer using whatever protocol you had available, requested a document and received text that you use for whatever you may have wanted*

### Extensibility
- The system was simple but it also needed to be extensible. The WWW was designed to exist for a long time, so it was necessary to make it capable of evolving over time. *Talking specifically about the HTTP Protocol, the power of extensibiliy can be seen with the hundreds of uses that HTTP headers have in the modern web*

### Distributed Hypermedia
- Distributed hypermedia allows documents to be stored in remote locations
- By its nature, distributed hypermedia required the transfer of large ammounts of data *4KB was a lot back in the 90s* so the WWW had to be designed for large-grain data transfer
- As the web's information resources are scattered allong the Internet, the WWW had to be designed to minimize network interactions and to minimize user-preceived latency

### Internet-scale
- As the web is intended to be an **Internet-scale distributed hypermedia system**, the suppliers of information services had to cpe with the demands of anarchic scalability and the independent deployment of software components

#### Anarchic Scalability
- Most systems are created with the assumption that the entire system is under the control of one entity, or that at least all entries participating in the system are acting towards a common goal. As the WWW is connected to the internet, this assumption cannot be made with this system.

- Anarchic Scalability refers to the need for elements of the WWW to be able to continue working even if subjected to an unanticipated load or when given malformed or maliciously constucted data, as they may be communicating with someone outside of their control.

- The architecture must be designed to enhance visibility and scalability

- Clients cannot be expected to maintain knowledge of all servers and servers cannot be expected to retain knowledge of state accross requests *HAA sessions defeated this requiremeent, but to be fair, sessions were implemented in a way where anarchic scalability wasn't a problem and where the state can be trusted*

- Hypermedia data elements cannot retain back-pointers that identify data elements that reference them, because if they did, they'd have to keep up with a ridiculous ammount of information, specially for really popular data elements.

- By the nature of the internet and anarchic scailability, participants in an application interaction must assume that any information received is untrusted, or require some additional authentication before trust can be given. This means that the WWW architecture must be able of communicating authentication data and authorization controls and that the default state of an application before any authentication has happened must be a limited version where no trusted data is needed. 

*This section pretty much acknowledges that when using the Internet as a plataform, you never know who might be on the other side of the communication and what their intentions might be. The WWW architecture tries to cover all of its bases by guarding itself against incorrect or malicuious data.*

*The WWW also defines that as you never know who's on the other side, trust must never be given without veryfing the other user's identity. This design consideration can still be seen in things from the modern Web like CORS. Also, funnily enought the earliest and most common types of web attacks (like XSS, SQL injection or CSRF) originate from an assumed trust in the other side of the connection and all of these attacks are resolved by discarding that assumed trust and verifying and protecting oneself against malicious or malformed data.*
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
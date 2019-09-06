---
layout: post
title:      "RESTful Routes"
date:       2019-09-06 09:24:33 -0400
permalink:  restful_routes
---


We're currently learning Sinatra and bringing together many of the concepts we've learned so far. RESTful routes have been one of the elegant web design elements we've learned this week. The REST (representational state transfer) principles provide a standardized mapping between HTTP verbs (get, post, patch, delete) and CRUD actions (create, read, update, delete). With RESTful routes we have standardized URLs, and delelopers have a streamlined pattern they can follow when creating websites. 

<strong>The 7 RESTful Routes</strong>

<table>
<tr>
<th>Action</th>
<th>Route</th>
<th>HTTP Verb</th>
<th>Description</th>
</tr>
<tr>
<td>Index</td>
<td>/cats</td>
<td>GET</td>
<td>Displays a list of all cats</td>
</tr>
<tr>
<td>New</td>
<td>/cats/new</td>
<td>GET</td>
<td>Displays a form to make a new cat</td>
</tr>
<tr>
<td>Create</td>
<td>/cats</td>
<td>POST</td>
<td>Adds a new cat to the database, then redirects to that cat's page ("/cats/:id")</td>
</tr>
<tr>
<td>Show</td>
<td>/cats/:id</td>
<td>GET</td>
<td>Shows information on the cat whose ID matches the ID in the URL</td>
</tr>
<tr>
<td>Edit</td>
<td>/cats/:id/edit</td>
<td>GET</td>
<td>Displays a form to edit the cat whose ID matches the ID in the URL</td>
</tr>
<tr>
<td>Update</td>
<td>/cats/:id</td>
<td>PATCH</td>
<td>Modifies the cat whose ID matches the ID in the URL, then redirects to that cat's page ("/cats/:id")</td>
</tr>
<tr>
<td>Delete</td>
<td>/cats/:id</td>
<td>DELETE</td>
<td>Deletes the cat from the database, then redirects to the index of all cats ("/cats")</td>
</tr>

</table>

By following these conventions, other developers can easily understand our URL pattern, and we don't have to invent a structure with every project. 

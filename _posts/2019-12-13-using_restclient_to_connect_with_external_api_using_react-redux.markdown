---
layout: post
title:      "Using RestClient to connect with external API using React-Redux"
date:       2019-12-13 16:28:33 +0000
permalink:  using_restclient_to_connect_with_external_api_using_react-redux
---


I am currently creating a web app that allows users to browse NYTimes bestelling books and add them to their list to read later. This required connecting the frontend to the [NYTimes Books API](https://developer.nytimes.com/apis). When the React frontend loads, it sends a request to the Rails backend, which handles fetching books from the NYTimes API.

## Set up React, Redux, and Thunk middleware

First, set up your `index.js` to create your redux store (so you can save books to the store) and import Thunk middleware (we'll use this when we set up fetch actions).

```
bookshelf-frontend/src/index.js:
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import manageBooks from "./reducers/manageBooks"

import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import thunk from "redux-thunk"

const store = createStore(manageBooks, applyMiddleware(thunk))

ReactDOM.render(
    <Provider store={store}>
      <App />
    </Provider>,
  document.getElementById('root')
)  
```

## Fetch the genre names and store them using Redux

### Initiate the fetch from the frontend

Next, we will fetch the list of genre names from the NYTimes API. The GenresContainer component is responsible for fetching the genre names and saving them in the global store using Redux. The genre names are then passed to a child component, Genres, that displays the genre names.

```
bookshelf-frontend/src/containers/GenresContainer.js:

import React, { Component } from 'react';
import { connect } from 'react-redux';
import Genres from '../components/Genres';
import { fetchGenres } from "../actions/nytimes";

class GenresContainer extends Component {

    componentDidMount() {
        this.props.fetchGenres();
    }

    render() {
        return (
            <div>
                <Genres fetchGenreBooks={this.props.fetchGenreBooks} genres={this.props.genres} />
            </div>
        )
  }
}

const mapStateToProps = ({ genres }) => ({ genres})

export default connect(mapStateToProps, { fetchGenres })(GenresContainer);
```

When the GenresContainer is mounted, it executes the `fetchGenres` action from `bookshelf-frontend/src/actions/nytimesjs:`

```
export const SET_GENRES = "SET_GENRES"
export const FETCH_GENRES = "FETCH_GENRES"


// Sets genres on the page after they're fetched
export const setGenres = genres => {
    return { type: SET_GENRES, genres };
};

// Fetches list of genres from NYTimes
export const fetchGenres = () => {
    return dispatch => 
        fetch('http://localhost:3000/genres')
            .then(response => response.json())
            .then(data => {
                dispatch(setGenres(data))
            });
}
```

The `fetchGenres` action tells the Rails backend to fetch the genres from the NYTimes API. This way the backend handles saving my NYTimes API key and making all API requests. I'll cover the Rails backend code below. Note that Thunk middleware needs to be imported to the index in order for the fetchGenres action to return a dispatch function (otherwise you can only return an object). We already set that up in `index.js`.

*Now let's move to the Rails backend to handle fetching from the NYTimes API*

### Fetch from the NYTimes API from the Rails backend

First, create a controller to handle the GET request to /genres. I called my controller BestsellersController:
```
bookshelf-api/app/controllers/bestsellers_controller.rb
require 'rest-client'

class BestsellersController < ApplicationController
    def genres
        genres_response = RestClient.get("https://api.nytimes.com/svc/books/v3/lists/names.json?api-key=#{ENV['NYTIMES_KEY']}")

        genres_data = JSON.parse(genres_response)["results"]

        @genres = genres_data.map do |genre|
            genre["list_name"]
        end.sort

        render json: @genres
    end

end
```
This uses RestClient to make the call to the NYTimes API, grab the genre names, then convert them into JSON. For each genre, I only need the genre name (called "list-name" in their API), so I create an array of those names. I then `render json: @genres` so that when this action is called, it displays the array of genre names in JSON. 

Next up, set up your route so you can make GET requests to http://localhost:3000/genres:
```
bookshelf-api/config/routes.rb
Rails.application.routes.draw do
  resources :books

  root to: 'books#index'
  get '/genres', to: 'bestsellers#genres'

end
```

*Okay -- back to the frontend to display those genre names*

## Display genre names
The backend will send the data back to us in JSON,  so then we call a second action, `setGenres` to actually display the genres on the page. That action is sent to the reducer, which returns the new redux state with the updated list of genres:

```
bookshelf-frontend/src/reducers/manageBooks.js:

import {
    SET_GENRES,
} from "../actions/nytimes";

export default function manageBooks(state = { userBooks: [], books: [], genres: [] }, action) {
    switch (action.type) {
        case SET_GENRES:
            return {...state, genres: [...action.genres]}

        default:
            return state;
    }
}
```

Through `mapStateToProps`, `GenresContainer` has access to these genres. These genres are then passed down to the `Genres` component in the render method (`<Genres fetchGenreBooks={this.props.fetchGenreBooks} genres={this.props.genres} />)`.

The `Genres` component is responsible for displaying the list of genres on the page:
```
bookshelf-frontend/src/components/Genres.js:
import React, { Component } from 'react';
import { Link } from 'react-router-dom';

class Genres extends Component {

    render() {
        return(
            <div className="container">
                <h3>Select a genre to see bestsellers</h3>
                <div className="col-15">
                    <div className="d-flex flex-row flex-wrap"> 
                    {this.props.genres.map(genre => 
                        <Link to={`/bestsellers/${genre}`} className="list-group-item w-50 list-group-item-action">{genre}</Link>
                    )}
                    </div>
                </div>
            </div>
        );
    }
};

export default Genres;
```

This generates the following view, where each genre name is a link to http://localhost:3001/bestsellers/:genre:

<img src="https://i.imgur.com/NxEFmPi.png" width="75%" >

## Set up Routes so each genre has its own route

We need to make sure that ./bestsellers/:genre is a valid route, so back in `App.js` we'll set up our routes:
```
bookshelf-frontend/src/App.js:
import React from 'react';
import {
  BrowserRouter as Router,
  Route
} from 'react-router-dom';
import './App.css';
import GenresContainer from './containers/GenresContainer'
import BooksContainer from './containers/BooksContainer'

function App() {
  return (
    <Router>
      <div className="App">
        <NavBar />
        <Route exact path="/" render={() => 
          <div>
            <h2>Bookshelf</h2>
            <h4>Browse NY Times Bestsellers and add them to your list to read later!</h4>
          </div>}
        />
        <Route exact path="/bestsellers" render={routerProps => <GenresContainer {...routerProps}/>}/>
        <Route path="/bestsellers/:genre" render={routerProps => <BooksContainer {...routerProps}/>}/>
      </div>
    </Router>
  );
}

export default App;
```

We set it up so that when a user visits /bestsellers/:genre, the BooksContainer will be displayed. The BooksContainer is responsible for the second phase of fetching from the NYTimes API. It is responsible for fetching all books associated with that particular genre, and it passes that book info down to the Book component to display each book. 

*Stay tuned -- I'll show how to set that up my next post!
*

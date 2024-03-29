# http-requests-API

## Establishing a connection between frontend and backend

1. The backend server should also be up and running. You do this with the locally installed backend server by running the node backend api in its own terminal window

- Install node .pkg Arm64 from nodejs.org
- cd backend
- npm install
- node app.js
- no command propmt return so it is running

2. Since the data is not available instantaly like when one uses: const places = localStorage.getItem("places") one must first run the component and then update it with the available places afterwards by setting a useState and returning the state as availablePlaces after it became available:
   import Places from "./Places.jsx";
   import { useState } from "react";

```
import Places from "./Places.jsx";
import { useState } from "react";

export default function AvailablePlaces({ onSelectPlace }) {
  const [availablePlace, setAvailablePlace] = useState([]);

  return <Places title="Available Places" places={availablePlaces} fallbackText="No places available." onSelectPlace={onSelectPlace} />;
}
```

3. Getting the data can be done in various ways

- fetch() built into the browser
- can be used to send or fetch an http request to another server
- fetch() at least wants the url of the server you want to send the request to
- In the server we want to target one of the endpoints provided there. In this project it is /places inside app.js

  ```Inside app.js:
  app.get('/places', async (req, res) => {
  const fileContent = await fs.readFile('./data/places.json');

  const placesData = JSON.parse(fileContent);

  res.status(200).json({ places: placesData });
  });
  ```

```Inside AvailablePlaces.jsx
- fetch("http://localhost:3000/places");
```

- This will send a get request to the url that will return a promise which is a javascript value that will eventually resolve to another value. So it is basically a wrapper object around a value that is not there yet (in this case an eventually received response object)
- To access such values one can change methods on the rusult of calling fetch. .then is one such method which contains a function that will execute once the promise has been received
  fetch("http://localhost:3000/places").then((response) => {});
- You can also use the await keyword if the component functions is preceded with the async keyword, but THIS IS NOT ALLOWED FOR COMPONENT FUNCTIONS.
  export default function async AvailablePlaces({ onSelectPlace }) {
  const [availablePlaces, setAvailablePlaces] = useState([]);

  const response = await fetch("http://localhost:3000/places").then((response) => {});

  return <Places title="Available Places" places={availablePlaces} fallbackText="No places available." onSelectPlace={onSelectPlace} />;
  }

- So we are using fetch. the response object is in this case json data. the ,json method can be used now on the response object to extract such json data.
  fetch("http://localhost:3000/places").then((response) => {
  response.json();
  });
- The json method returns another promise so if we return json we can add another then method after the first one to eventually get back our response data and work with that data
  fetch("http://localhost:3000/places")
  .then((response) => {
  return response.json();
  }).then((resData) => {});
- This is now our final response data which in our case is the places object that has a places key with the placesData as the value for an array of available places AS EXPECTED BY THE GET ENDPOINT FOR PLACES
  app.get('/places', async (req, res) => {

      const fileContent = await fs.readFile('./data/places.json');

      const placesData = JSON.parse(fileContent);

      res.status(200).json({ places: placesData });

  });

Using fs.readFile, the server reads the content of ./data/places.json. The fs module is a part of Node.js's standard library, and readFile is an asynchronous function that reads the contents of a file.

Parses the JSON content: The content of the file is in JSON format (a string), so JSON.parse is used to convert this string into a JavaScript object. This is necessary because while JSON is a text format that looks like an object, it needs to be parsed to be used as an actual JavaScript object in code.

Sends a response: Finally, the server sends a response back to the client with a status code of 200 (indicating success). The response body is in JSON format, created using res.json(). This method converts a JavaScript object or array into a JSON string and sets the appropriate Content-Type header (application/json). Here, the object { places: placesData } is sent back, where placesData is the JavaScript object obtained from parsing the JSON file content.

- Now we can get access to this object with
  fetch("http://localhost:3000/places")
  .then((response) => {
  return response.json();
  })
  .then((resData) => {
  resData.places
  });
- And now we can call setAvailablePlaces and pass this data to it:
  fetch("http://localhost:3000/places")
  .then((response) => {
  return response.json();
  })
  .then((resData) => {
  setAvailablePlaces(resData.places);
  });
  - This code now would create an infinite loop. That is why a get request gets wrapped with useEffect in react - to prevent the loop

## Using async await

- To make the code more readable you can replace the .then methods with the async await structure inside the useEffect function. This is allowed because we are now working with code that is created by us and not on component level where react expect specific form

  ```
  useEffect(() => {
  async function fetchPlaces() {
  const response = await fetch("http://localhost:3000/places");
  const resData = await response.json();
  setAvailablePlaces(resData.places);
  }

  fetchPlaces();
  }, []);
  ```

  - Note: The async is never put in front of the () in the arrow function
  - fetchPlaces() needs to be executed outside the fetch function

## throtteling - slow 3d to test where to put loading scripts

- Inside your network tab insdie developer tools you can select slow 3d to throttle the data and you'll see how it takes a while for the pictures to arrive. This is not a great user experience. It would make sense for some fallback text to be displayed while we are waiting for the data to arrive.
- We allready have fallback text for the situation where we do not have pplaces available, but not for where the places/pics are just slow to arrive, because they have not be fetched yet. So you can set this prop in AvailablePlaces.jsx
  loadingText="Fetching place data..."
- And the also a prop that shows true for while we are waiting for the places to arrive and false when the places have been fetched and the data is there
  isLoading={true}
- Inside Places.jsx you should now accept these two props and then use these two props to show some special output. Note the two new condition adjustments: One new one set for when isLoading is true and two places where isLoading is false and no new places is as well not available(!isLoading && places.length === 0 )

```
export default function Places({ title, places, fallbackText, onSelectPlace, isLoading, loadingText }) {
console.log(places);
return (
  <section className="places-category">
    <h2>{title}</h2>
    {isLoading && <p className="fallback-text">{loadingText}</p>}
    {!isLoading && places.length === 0 && <p className="fallback-text">{fallbackText}</p>}
    {!isLoading && places.length > 0 && (
      <ul className="places">
        {places.map((place) => (
          <li key={place.id} className="place-item">
            <button onClick={() => onSelectPlace(place)}>
              <img src={`http://localhost:3000/${place.image.src}`} alt={place.image.alt} />
  <h3>{place.title}</h3>
  </button>
  </li>
  ))}
  </ul>
  )}
  </section>
  );
  }
```

- isLoading is now set to true, but it should really be set synamically through our http request. As long as the data is on its way it should be set to true and when it has arrived it should be set to false
- To achieve this you need to set some state insidte the AvailablePlaces component and set it to true inside the useEffect function or inside the fetchPlaces function and to false once we have the data

  useEffect(() => {
  //setIsFetching(true);
  async function fetchPlaces() {
  setIsFetching(true);
  const response = await fetch("http://localhost:3000/places");
  const resData = await response.json();
  setAvailablePlaces(resData.places);
  setIsFetching(false)
  }

  fetchPlaces();
  }, []);

- So now you can use the isFetching state as a value for the isLoading prop

## Fetching data might fail resulting in errors

- Data fetching can fail in two ways and one needs to cater for both cases:
  a. You might fail to send the get request in the first place like where th network connection crash
  b. Something goes wrong in the backend which sends an error response
- Error response situation:

  1. First check for the response.ok property
  2. When - if(response.ok) is true it will deliver a status code of 200 or 200
  3. When - if(response.ok) is false it will deliver a status code of 400 or 500
  4. If it is false you want to deal with the error situation by for example throwing an error by using the built in Error class

     if (!response.ok) {
     const error = new Error("Failed to fetch places");
     throw error
     }

  or do it all on one step:

      if (!response.ok) {
        throw new Error("Failed to fetch places");
      }

  5.  Throwing an error will crach the appliction. You can prevent this by wrapping your code into a try catch block around the code that might fail and then catch the error that might be thrown and then define some code that should be executed if an error was encountered.
  6.  Other than yourself writing code that throws an error, the fetch function can throw an error all by itself if there is for instance no connection. So the try catch block is precisely what we want here because either way we end up in the catch block where we can handle the error when anything goes wrong.

  useEffect(() => {
  //setIsFetching(true);
  async function fetchPlaces() {
  setIsFetching(true);

      try {
        const response = await fetch("http://localhost:3000/places");
        const resData = await response.json();

        if (!response.ok) {
          throw new Error("Failed to fetch places");
        }
      } catch (error) {
        // code...
      }

      setAvailablePlaces(resData.places);
      setIsFetching(false);

  }

  fetchPlaces();
  }, []);

  7. Typically handling the error in a react application means that we want to update the UI and show an error message to the user. This means that we will need a piece of state / and error state. It is supercoming to have these three pieces of state together when fetching data.

  - the data state where you store the data that you are fetching (availablePlaces)
  - the loading state where you tell the user you are currently fetching data (isFetching)
  - the error state which you use to show potential errors

  8. So now we can setError(error) in the catch block and set the caught error as the value for the error state
  9. The problem we have with the data as it is setup now is that we are trying to set the resData outside the try block. If an error results from the try block the resDate can never have data available to be set on the setAvailablePlaces. So the setAvailablePlaces must be moved into the try block

10. If an error is encountered you can return a separate error component. So call this error component and then set the required props for title and message

if (error) {
return <Error title="An error occured" message={error.message} />;
}

and create a fallback message inside the catch block

} catch (error) {
setError({ message: error.message || "Could not fetch places, please try again later" });
}

## Handling data before you use it in the UI

- Lets say we want to sort the response object before it is displayed inside the browser
- Our project has a loc sort file that will calculate the distance between myself and the destination place and sort the places accordingly from nearest to furthest.
- The navigator object that is provided by the browser can fetch the location of the user.
  navigator.geolocation.getCurrentPosition()
- The position is not available instantly, therefore getCurrentPosition() takes a function that will be executed by the browser once the data becomes available. One cannot use async await here because this method does not yield a promise, but rather use a call back pattern that defines the code that should be executed once the position coordinates have been fetched. So the postion coordinates are received inside this function and it is here that you should set your calculations by taking this position in

        navigator.geolocation.getCurrentPosition((position) => {
          const sortedPlaces = sortPlacesByDistance(resData.places, position.coords.latitude, position.coords.langitude);
        setAvailablePlaces(sortedPlaces);
        });

- A problem now is that since we are not running a async await function here, but a callback, js will execute the setIsFetching(false) line right after the getCurrentPosition has been invoked, but before the data is received back. So one should move that part of the code into the callback function after setting the available places AND one should also call after an error is returned if there is one.
  import { useEffect, useState } from "react";
  import Places from "./Places.jsx";
  import Error from "./Error.jsx";
  import { sortPlacesByDistance } from "../loc.js";

export default function AvailablePlaces({ onSelectPlace }) {
const [isFetching, setIsFetching] = useState();
const [availablePlaces, setAvailablePlaces] = useState([]);
const [error, setError] = useState();

useEffect(() => {
//setIsFetching(true);
async function fetchPlaces() {
setIsFetching(true);

      try {
        const response = await fetch("http://localhost:3000/places");
        const resData = await response.json();

        if (!response.ok) {
          throw new Error("Failed to fetch places");
        }

        navigator.geolocation.getCurrentPosition((position) => {
          const sortedPlaces = sortPlacesByDistance(resData.places, position.coords.latitude, position.coords.langitude);

          setAvailablePlaces(sortedPlaces);
          setIsFetching(false);
        });
      } catch (error) {
        setError({ message: error.message || "Could not fetch places, please try again later" });
        setIsFetching(false);
      }
    }

    fetchPlaces();

}, []);

if (error) {
return <Error title="An error occured" message={error.message} />;
}

return <Places title="Available Places" places={availablePlaces} isLoading={isFetching} loadingText="Fetching place data..." fallbackText="No places available." onSelectPlace={onSelectPlace} />;
}

## Extracting code to simplify

- By moving the fetch part from inside the try block to a utility function you can either throw the error if there is one or you can return the resData.places if there is data
- So now inside the Available places you need to import this function back into the AvailablePlaces component and call it inside the try blockm and now you can await it because it would deliver a promise.
- put this into a const that you can now use in the place of resDate.places
- potentially this outcourced code can now be used in other places in the app
  useEffect(() => {
  async function fetchPlaces() {
  setIsFetching(true);

        try {
          const places = await fetchAvailablePlaces();

          navigator.geolocation.getCurrentPosition((position) => {
            const sortedPlaces = sortPlacesByDistance(places, position.coords.latitude, position.coords.langitude);

            setAvailablePlaces(sortedPlaces);
            setIsFetching(false);
          });
        } catch (error) {
          setError({ message: error.message || "Could not fetch places, please try again later" });
          setIsFetching(false);
        }

  }

  fetchPlaces();
  }, []);

  - Code for the outsourced http request
    export async function fetchAvailablePlaces() {
    const response = await fetch("http://localhost:3000/places");
    const resData = await response.json();

  if (!response.ok) {
  throw new Error("Failed to fetch places");
  }
  return resData.places;
  }

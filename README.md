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

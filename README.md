# Structured Literature Review App

This is a web application built using Vue.js that enables users to classify and label input data obtained from an API. The app is designed to provide support for structured literature reviews.

### Like this? Have a look at:

[<img src="https://dtdi.de/ads/paper-swipe.png" width="419px" />](https://dtdi.de/i.php?repo=paper-swipe)

## Features

- Allows users to classify and label input data
- Fetches data from an API
- Categorizes data based on the user's interests
- Provides an easy-to-use interface for quick categorization
- The user can filter data based on the year
- Allows the user to save an API token

## Getting Started

To run the project locally, follow these steps:

1. Clone the repository
2. Customize code for your lit rev.
3. Install the dependencies by running `npm install`
4. Start the development server using `npm run serve`

The app should now be accessible at `http://localhost:8080`.

## Customization

in your .env file:

```
VITE_ENDPOINT=[Your NocoDB Instance]
VITE_NOCODB_QUERY="sort=-Year,Title&where=(RelevantTf,eq,1 - mid)&limit=40&shuffle=0"
VITE_HIGHLIGHT_REGEX="/business process\w*|design\w*|redesign\w*|improv\w*|reengin\w*|re-engin\w*|improv\w*|assist\w*|enhanc\w*|change\w*|innovat\w*|optimi\w*|automat\w*|support\w*|optimi\w*|machine learning\w*|machine-learning\w*|system\w*|approach\w*|method\w*|algorithm\w*|tool\w*/"
```

## Usage

When the app is launched, it will load data from the API and present the first item for the user to classify. The user can then classify the item by clicking on the relevant button or by using keyboard shortcuts. The user can filter the data based on the year by entering the year in the input field at the top of the app. The user can also save an API token to avoid having to enter it every time the app is launched.

## Acknowledgments

- This project was built using Vue.js and several other open-source libraries.
- The API used in this project is provided by Noco DB.

## Contributing

Contributions are welcome! Please feel free to submit a pull request or open an issue if you find a bug or have a feature request.

## 🙇 Author

#### Tobias Fehrer

- Twitter: [@dtdi\_](https://twitter.com/dtdi_)
- Github: [@dtdi](https://github.com/dtdi)

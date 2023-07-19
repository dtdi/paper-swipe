# Structured Literature Review App

This tool is designed to facilitate the rapid classification of a list of items based on predetermined criteria - similar to swiping in Tinder or using simple key inputs on a PC. The tool doesn't find new papers but aids in swiftly sorting through existing ones.

During one project, I used it to classify around 10,000 entries. The tool operates as a website, so it can be used from anywhere (I've found myself classifying entries while waiting in line at the bakery).

### Like this? Also have a look at:

[<img src="https://dtdi.de/ads/paper-swipe.png" width="419px" />](https://dtdi.de/i.php?repo=paper-swipe)

## Features

- Allows users to classify and label input data
- Fetches data from an API
- Categorizes data based on the user's interests
- Provides an easy-to-use interface for quick categorization
- The user can filter data based on the year
- Allows the user to save an API token

## Data Preparation

If you want to use the app, i am more than happy to help with the setup. I'll need your items, classification criteria, and a few app settings.

### Item List Structure

Please provide your list of items, ideally in an Excel spreadsheet with the following columns:

- `title`
- `abstract`
- `year`
- `id`: This can be a simple incremental number, but it will help you later as a reference for further work. We have had good experience with a 3-letter abbreviation of the search result provider + continuous number with padding. For example, `WOS0001` for the first entry from our Web of Science search.
- `type` (optional): for example, journal or conference
- `source` (optional): name of the outlet / conference

### Classification Structure

For example, your classifications could look like this:

| Relevant | Artefact             | Focus           |
| -------- | -------------------- | --------------- |
| 0 - no   | 0 - no               | 0 - no          |
| 1 - mid  | 1 - yes              | 1 - bpr         |
| 2 - yes  | 2 - yes instantiated | 2 - design time |

Most classifications are optional. I've set up the tool to classify 'Relevant No' when swiping left and 'Relevant Yes' when swiping right. Your classification is saved as soon as you swipe or click one of the 'Relevant' buttons.

### App Settings

- **Keywords**: Any keywords you want highlighted. Like in searches, we can work with subwords here.
  - For example: `business process\w*|design\w*|redesign\w*|improv\w*|reengin\w*|re-engin\w*`
- **Sort Order**: How you would like your items sorted.
  - For example, descending by year.
- **Classification strategy**: Do you want both of you to separately classify all papers, or is it sufficient if each of you looks at a paper?
  - Depending on your choice, you'll get one or two fields for each classification category. The Excel file you can export from the tool afterwards allows you to easily identify conflicts and adjust accordingly.

## Getting Started with the code

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

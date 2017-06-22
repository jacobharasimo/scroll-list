# Scroll a Big List

We would like to display a list of 500k employees of a big corporation in a single list on a page (SPA). The `update()` function below accepts a list of indicies (positions of an employee in a list) and makes a request to the server, updating the data in a local store.

You can see a live example of the app (using React) running at [flexitive.github.io](https://flexitive.github.io/engineering-interviews/scroll-list/example/).

Expand/update the code below to **optimize for performance**. Below are some pointers to get you started. It’s better to address a few of these points fully, rather that to produce an incomplete solution to all of them. We are looking for code, not prose - you will have a chance to explain your design solutions on-site.

1. How can we **cache the data we fetch from the server locally** to avoid requesting the same pieces of information over and over?
    * React uses a store. Each time you go to fetch data since you have the `indicies` check the `store.list` to see if it has that `indicies`. If not fetch it otherwise return. Likewise in these cases no loading animation is needed for the `Row`
2. What about requests where the resulting data are/will **no longer be needed**. How would we skip/cancel those?
    * not implemented: since we are already watching the scroll when an item scrolls off the page remove it from the `requestQue` if it exists.
3. How can we make sure we don’t make **too many requests** at the same time, making sure that we make say at most 1 request per 500ms? 
    * when you fetch data `indicies` are added to an array. a timer then starts to fetch the data. if another request happens it will clear any pending timers, add items to the arrray and start a new timer. This will insure that data fetch happens only once every 500ms. Once the data is retrieved it removes the items from the array of `indicies`.
4. How can we collect the requests for data and send them in **batches**?
    * each time we call update we push the `indicies` to the `requestQue` if we don't already have it. once the data fetch runs it gets all the items from the `requestQue` and then removes them 
5. How would we handle a **loading state** of an employee row?
    * for each row we can have an independent loader. This loader defaults to isLoading = true and once the component is rendered we then show the row. For UX reasons we add a timeout to this to prevent a 'flicker' for fast loading items
6. What if we want to **limit** the number of items we want to keep/cache locally to say 200 items?
    * not implemented: When you add an item to the store check for the array length. If the array is 200 items long for every item you add to the array pop one off the top
7. What if we **scroll** a large chunk of the list say every 200ms, never getting to see the data that we get back. How could we cancel requests like this?
    * not implemented: since we are already watching the scroll when an item scrolls off the page remove it from the `requestQue` if it exists.

Use any library you feel appropriate.

```javascript
const store = {
  offset: 0,
  size: 500,
  list: [],
};

class Row extends React.Component {
  componentDidMount() {
    const { i, list } = this.props;
    if (list[i]) return;
    this.props.update([ this.props.i ]);
  }

  render() {
    const { i, list } = this.props;
    return <div className="row">{i}: {list[i]}</div>;
  }
}

class List extends React.Component {
  constructor() {
    super();
    this.state = store;
    this.update = this.update.bind(this);
  }

  componentDidMount() {
    window.addEventListener('scroll', () => {
      const offset = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop || 0;
      this.setState({ offset });
    });
  }

  update(indicies) {
    app.fetch(indicies).then((res) => {
      indicies.forEach((index, i) => {
        store.list[index] = res[i];
        this.setState({ list: store.list });
      });
    });
  }

  render() {
    const { offset, size, list } = this.state;

    const count = Math.ceil(window.innerHeight / app.rowHeight);
    let a = Math.floor(offset / app.rowHeight);
    let b = Math.max(0, size - a - count);

    const rows = _.map(_.range(Math.min(count, size - a)), (i) => {
      i+= a;
      return <Row key={i} i={i} list={list} update={this.update} />;
    });

    return (
      <div>
        <div style={{ height: `${a * app.rowHeight}px` }} />
        <div>{rows}</div>
        <div style={{ height: `${b * app.rowHeight}px` }} />
      </div>
    );
  }
}
```

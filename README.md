# react-refactoring-code-smells

## Table of contents

- [Introduction](#introduction)
- [Refactoring techniques](#refactoring-techniques)
    - [Extract component](#extract-component)
    - [Extract logic to a custom hook](#extract-logic-to-a-custom-hook)
    - [Remove unused props](#remove-unused-props)
    - [Split component](#split-component)
    - [Extract HTML/JS code to component](#extract-htmljs-code-to-component)
    - [Extract JSX outside render method to component](#extract-jsx-outside-render-method-to-component)
    - [Remove props in initial state](#remove-props-in-initial-state)
    - [Extract higher order component](#extract-higher-order-component)
    - [Remove direct DOM manipulation](#remove-direct-dom-manipulation)
    - [Remove force update](#remove-force-update)
    - [Extract logic to a custom context](#extract-logic-to-a-custom-context)

## Introduction

WIP

## Refactoring techniques

### Extract component

This refactoring is analogous to the traditional _Extract Class_ refactoring, and can occur when parts of a component appear in multiple places. Therefore, this refactoring consists in extracting these parts into a new component, allowing their reuse in other places. In the code below, the `Post` component and the `Story` component share the same actions buttons:

```jsx
function Post({ post }) {
  return (
    <div className="post">
      <h2>{ post.title }</h2>
      <p>By { post.author } on { post.date }</p>
      <p>{ post.content }</p>
      <div className="comments">
        { post.comments.map((comment) => (
          <div key={ comment.id }>
            <p>{ comment.text }</p>
            <p>By { comment.author }</p>
          </div>
        )) }
      </div>
      <div className="actions">
        <button>Like</button>
        <button>Comment</button>
        <button>Share</button>
      </div>
    </div>
  );
}

function Story({ story }) {
  return (
    <div className="story">
      <img src={story.src} alt={story.alt}>
      <div className="actions">
        <button>Like</button>
        <button>Comment</button>
        <button>Share</button>
      </div>
    </div>
  );
}
```

In the code below, the refactoring was applied, extracting the new `PostActions` component:

```jsx
function PostActions() {
  return (
    <div className="actions">
      <button>Like</button>
      <button>Comment</button>
      <button>Share</button>
    </div>
  );
}

function Post({ post }) {
  return (
    <div className="post">
      <h2>{ post.title }</h2>
      <p>By { post.author } on { post.date }</p>
      <p>{ post.content }</p>
      <div className="comments">
        { post.comments.map((comment) => (
          <div key={ comment.id }>
            <p>{ comment.text }</p>
            <p>By { comment.author }</p>
          </div>
        )) }
      </div>
      <PostActions />
    </div>
  );
}

function Story({ story }) {
  return (
    <div className="story">
      <img src={story.src} alt={story.alt}>
      <PostActions />
    </div>
  );
}
```


### Extract logic to a custom hook

React hooks were introduced in React 16.8 to use state and other features without needing class components. However, the logic that deals with state might become duplicated in components. This refactoring consists of implementing a custom hook with the target logic, and replace the current logic in the component with the new hook. In the code below, there is an API request logic using three state variables:

```jsx
function ExampleComponent() {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      try {
        const response = await fetch("https://api.example.com/data");
        const data = await response.json();
        setData(data);
      } catch (error) {
        setError(error);
      } finally {
        setIsLoading(false);
      }
    };

    fetchData();
  }, []);

  return (...);
}
```

In the code below, the refactoring was applied. The `useFetchData` hook was created, and the API request logic was moved to it. Now, the `ExampleComponent` uses the new hook, and other components can use it too, without duplicating the code:

```jsx
function useFetchData(url) {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      try {
        const response = await fetch(url);
        const data = await response.json();
        setData(data);
      } catch (error) {
        setError(error);
      } finally {
        setIsLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, isLoading, error };
}

function ExampleComponent() {
  const {
    data,
    isLoading,
    error,
  } = useFetchData("https://api.example.com/data");

  return (...);
}
```


### Remove unused props

In React components, props are the parameters sent in the JSX code, like in `<Component prop1="value1" prop2="value2" />`. It is a common practice during the development to pass props to child components. However, as the application evolves, some props may become unused due to changes in the component’s logic or requirements. This refactoring is analogous to the traditional _Remove unused parameter_ and can occur when the component accepts a prop that is never used by it. Thereby, this refactoring consists in removing the unused props from the component signature. The code below represents an example of this problem, as the `color` prop is never used in the component:

```jsx
function ProductCard({ name, price, color }) {
  return (
    <div className="product-card">
      <h3>{ name }</h3>
      <p>$ { price }</p>
    </div>
  );
}
```

In the code below, the refactoring was applied, removing the unused prop:

```jsx
function ProductCard({ name, price }) {
  return (
    <div className="product-card">
      <h3>{ name }</h3>
      <p>$ { price }</p>
    </div>
  );
}
```


### Split component

This refactoring is analogous to the traditional _Extract Class_ refactoring, and can occur when a component starts getting too large, with many responsibilities, making it hard to maintain. That way, Therefore, this refactoring consists in separating the component into smaller ones. The code below represents a component with different responsibilities, having three targets for this refactoring:

```jsx
function ProductDetail({ product }) {
  return (
    <div className="product-detail">
      <h2>{ product.name }</h2>
      <p>Price: $ { product.price }</p>
      <h3>Details</h3>
      <ul>
        {product.specs.map((spec) => (
          <li key={spec.id}>{spec.name}: {spec.value}</li>
        ))}
      </ul>
      <h3>Reviews</h3>
      <ul>
        {product.reviews.map((review) => (
          <li key={review.id}>
            { review.rating } stars - { review.text }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

In the code below, the refactoring was applied, splitting the `ProductDetail` component into three new components: `ProductInfo`, `ProductSpecs`, `ProductReviews`.

```jsx
function ProductInfo({ name, price }) {
  return (
    <div className="product-info">
      <h2>{ name }</h2>
      <p>Price: $ { price }</p>
    </div>
  );
}

function ProductSpecs({ specs }) {
  return (
    <ul className="product-specs">
      {specs.map((spec) => (
        <li key={spec.id}>{spec.name}: {spec.value}</li>
      ))}
    </ul>
  );
}

function ProductReviews({ reviews }) {
  return (
    <ul className="product-reviews">
      {reviews.map((review) => (
        <li key={review.id}>
          { review.rating } stars - { review.text }
        </li>
      ))}
    </ul>
  );
}

function ProductDetail({ product }) {
  return (
    <div className="product-detail">
      <ProductInfo name={product.name} price={product.price} />
      <ProductSpecs specs={product.specs} />
      <ProductReviews reviews={product.reviews} />
    </div>
  );
}
```

### Extract HTML/JS code to component

In web apps, UI elements like buttons and inputs are often very similar, changing only details like text and color. The problem happens when the HTML/JS code that implements these elements is duplicated, making it more difficult to maintain, reuse, and evolve. Thus, this refactoring consists in extracting the repeatable HTML/JS code into a new component. In the code below, there are two components using the same button, with little differences:

```jsx
function ExampleComponent1() {
    const style = {
        backgroundColor: "blue",
        color: "white",
    };

    function handleClick(event) {
        ...
    }

    return (
        <button
            style={style}
            onClick={handleClick}
        >
            This is the component 1!
        </button>
    )
}

function ExampleComponent2() {
    const style = {
        backgroundColor: "white",
        color: "red",
    };

    function handleClick(event) {
        ...
    }

    return (
        <button
            style={style}
            onClick={handleClick}
        >
            This is the component 2!
        </button>
    )
}
```

In the code below, the refactoring was applied, extracting the common HTML/JS code into a new component:

```jsx
function Button(props) {
    const style = {
        backgroundColor: props.background,
        color: props.color,
    };

    return (
        <button
            style={style}
            onClick={props.onClick}
        >
            { props.text }
        </button>
    );
}

function ExampleComponent1() {
    return (
        <Button
            text="This is the component 1!"
            color="white"
            background="blue"
        />
    )
}

function ExampleComponent2() {
    return (
        <Button
            text="This is the component 2!"
            color="red"
            background="white"
        />
    )
}
```

### Extract JSX outside render method to component

In React, the `render()` method (which is the only method required in a class component) returns a JSX template describing what should appear on the UI. However, when this method becomes large, developers sometimes move part of its code to separate methods, which prevents reuse decoupled from the render. So, this refactoring consists in extracting the method with JSX code into a new component. In the code below, the component defines the method `itemsList()`, and uses it in the `render()` method to get the JSX of the list:

```jsx
class ExampleComponent extends Component {
    itemsList() {
        return (
            <ul>
                ...
            </ul>
        );
    }

    render() {
        return (
            <div>
                <h2>List of items</h2>
                { itemsList() }
            </div>
        )
    }
}
```

In the code below, the refactoring was applied, extracting the JSX code in the `itemsList()` into a new component:

```jsx
class ItemsList extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
            <ul>
                { this.props.items.map(item => <li>{ item }</li>) }
            </ul>
        );
    }
}

class ExampleComponent extends Component {
    render() {
        return (
            <div>
                <h2>List of items</h2>
                <ItemsList items={["Item 1", "Item 2"]} />
            </div>
        )
    }
}
```

### Remove props in initial state

Initializing the state of a component with its props makes it ignore all props update from the parent. If the props values change, the component renders its initial values. Hence, this refactoring consists in removing the props from the initialization of state, and using the props value directly. In the code below, when the `ParentComponent`'s button is clicked, the number displayed by the `ChildComponent` is not updated since the state was initialized with the component's props.

```jsx
function ChildComponent(props) {
    const [currentNumber, setCurrentNumber] = useState(props.current);

    return (
        <div>
            <p>{ currentNumber }</p>
        </div>
    );
}

function ParentComponent() {
    const [currentNumber, setCurrentNumber] = useState(0);

    function handleClick() {
        setCurrentNumber(currentNumber + 1);
    }

    return (
        <div>
            <ChildComponent current={currentNumber} />
            <button onClick={handleClick}>Increase</button>
        </div>
    );
}
```

In the code below, the refactoring was applied, removing the state initialization, and using the props value directly:

```jsx
function ChildComponent(props) {
    return (
        <div>
            <p>{ props.current }</p>
        </div>
    );
}

function ParentComponent() {
    const [currentNumber, setCurrentNumber] = useState(0);

    function handleClick() {
        setCurrentNumber(currentNumber + 1);
    }

    return (
        <div>
            <ChildComponent current={currentNumber} />
            <button onClick={handleClick}>Increase</button>
        </div>
    );
}
```

### Extract higher order component

A higher-order component (HOC) is an advanced technique in React for reusing component logic. It takes a component as an input and returns a new enhanced component. Thus, HOC can be used to wrap around other components and provide additional functionality or data to them. Thereby, this refactoring consists in extracting common functionality from multiple components into a HOC, avoiding code duplication and making it easier to manage and update the shared functionality. In the code below there are two components with similar functionalities. The `ClickComponent` updates a value when a button is clicked, and the `HoverComponent` updates a value when a button is hovered:

```jsx
function ClickComponent() {
    const [value, setValue] = useState(0);

    function handleClick() {
        setValue(value + 1);
    }

    return (
        <div>
            <p>{ value }</p>
            <button onClick={handleClick}>Click</button>
        </div>
    );
}

function HoverComponent() {
    const [value, setValue] = useState(0);

    function handleHover() {
        setValue(value + 1);
    }

    return (
        <div>
            <p>{ value }</p>
            <button onHover={handleHover}>Hover</button>
        </div>
    );
}
```

In the code below, the refactoring was applied, extracting the common logic into the new HOC `withCounter()`. Note that the derived components `ClickCounter` and `HoverCounter` need to be used instead of the original components, because the derived components were enhanced by the HOC, receiving the props by it.

```jsx
function withCounter(OriginalComponent) {
    function CounterComponent() {
        const [value, setValue] = useState(0);

        function handleAction() {
            setValue(value + 1);
        }

        return (
            <OriginalComponent
                counter={value}
                increment={handleAction}
            />
        );
    }

    return CounterComponent;
}

function ClickComponent(props) {
    return (
        <div>
            <p>{ props.value }</p>
            <button onClick={props.increment}>Click</button>
        </div>
    );
}

function HoverComponent(props) {
    return (
        <div>
            <p>{ props.value }</p>
            <button onHover={props.increment}>Hover</button>
        </div>
    );
}

const ClickCounter = withCounter(ClickComponent);
const HoverCounter = withCounter(HoverComponent);
```

### Remove direct DOM manipulation

React uses its own representation of the DOM, called virtual DOM. When the state changes, React updates the virtual DOM and propagates the changes to the real DOM. However, manipulating the DOM using standard JavaScript code can cause inconsistencies between React’s virtual DOM and the real DOM. Thereby, this refactoring consists in removing any direct DOM manipulation from the component. In the code below, an HTML element is retrieved from the DOM by the `document.getElementById()` function and its style is directly updated:

```jsx
function ExampleComponent() {
  function handleClick() {
    const element = document.getElementById("element");
    element.style.backgroundColor = "blue";
  };

  return (
    <div>
      <button onClick={handleClick}>Click</button>
      <div id="element">
        This box will turn blue
      </div>
    </div>
  );
}
```

In the code below, the refactoring was applied, replacing the DOM operation with a state variable. Now, the background color is saved, and sent to the element in its style prop:

```jsx
function ExampleComponent() {
  const [backgroundColor, setBackgroundColor] = useState("white");

  function handleClick() {
    setBackgroundColor("blue");
  };

  return (
    <div>
      <button onClick={handleClick}>Click</button>
      <div style={{ backgroundColor }}>
        This box will turn blue
      </div>
    </div>
  );
}
```

### Remove force update

In order to automatically reflect model changes in the view, React re-renders a component only if its state or the props passed to it changed. However, developers can force the update of components or even reload the entire page, which may cause inconsistencies between the model and view. Thus, this refactoring consists in removing `forceUpdate()` call in the component. In the code below, there is no prop being sent to the `MapComponent`, as it's totally responsible for the data fetching and rendering. So, the `forceUpdate()` method is used when the parent's button is clicked:

```jsx
class MapComponent extends Component {
    getData() {
        ...
    }

    render() {
        return (...);
    }
}

class ExampleComponent extends Component {
    handleClick() {
        this.forceUpdate();
    }

    render() {
        return (
            <div>
                <button onClick={this.handleClick}>
                    Update map
                </button>
                <MapComponent />
            </div>
        );
    }
}
```

In the code below, the refactoring was applied, removing the `forceUpdate()` call. Only this would break the component, so the button was moved to the `MapComponent`, allowing it to directly call the `getData` method, which will cause the component to re-render.

```jsx
class MapComponent extends Component {
    getData() {
        ...
    }

    render() {
        return (
            <div>
                <button onClick={this.getData}>
                    Update map
                </button>
                ...
            </div>
        );
    }
}

class ExampleComponent extends Component {
    render() {
        return (
            <MapComponent />
        );
    }
}
```

### Extract logic to a custom context

Typically, data is passed from parent to child components using props. However, this can become complex when dealing with deeply nested component structures or when multiple components require the same information. React's context provides a solution by allowing a parent component to share data with any child component, regardless of its depth, without the need for explicit prop passing. In the code below, there is a `theme` state being passed to all the component tree:

```jsx
function Component2(props) {
    return (
        ...
    );
}

function Component1(props) {
    return (
        <Component2
            theme={props.theme}
            setTheme={props.setTheme}
        />
    );
}

function App() {
    const [theme, setTheme] = useState("dark");

    return (
        <Component1
            theme={theme}
            setTheme={setTheme}
        />
    );
}
```

In the code below, the refactoring was applied, removing the props related to the theme, and implementing the new context `ThemeContext`. Now, any component in the component tree can access the state provided by the context without passing it around.

```jsx
const ThemeContext = createContext({
  theme: "dark",
  toggleTheme: undefined,
});

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("dark");

  const toggleTheme = (newTheme) => {
    setTheme(newTheme);
  };

  return (
    <ThemeContext.Provider
        value={{ theme, toggleTheme }}
    >
      {children}
    </ThemeContext.Provider>
  );
}

function Component2(props) {
    const { theme } = useContext(ThemeContext);

    return (
        ...
    );
}

function Component1() {
    return (
        <Component2 />
    );
}

function App() {
    const [theme, setTheme] = useState("dark");

    return (
        <ThemeProvider>
            <Component1 />
        </ThemeProvider>
    );
}
```
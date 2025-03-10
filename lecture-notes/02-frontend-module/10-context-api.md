# 10: Context API

## Preparation

Use the repository from the previous **Formative Assessment**. Create a new branch called `10-playground`. Checkout to the `10-playground` branch and open the repository in **Visual Studio Code**.

---

## Create React App

To get started, run the following commands:

```bash
npx create-react-app book-shop
cd book-shop
```

**Resource:** <https://create-react-app.dev>

---

## Book Shop

In the `root/src` directory of the `book-shop` application, create a new directory called `components`.

### src/components/Book.js

In the `components` directory, create a new file called `Book.js`. In the `Book.js` file, add the following code:

```jsx
const Book = (props) => {
  return (
    <>
      <h1>{props.name}</h1>
      <h3>${props.price}</h3>
      <button onClick={() => props.addToCart()}>Add to cart</button>
    </>
  );
};

export default Book;
```

**What is happening?**

A `Book` has three `props` - `name`, i.e., **Pride and Prejudice**, `price`, i.e., **$10** and `addToCart()`, i.e., add an object containing `name` and `price` to an array. This data will be passed down from the `BookList.js` file or `BookList` **component**.

### src/components/BookList.js

In the `components` directory, create a new file called `BookList.js`. In the `BookList.js` file, add the following code:

```jsx
import { useEffect, useState } from "react";
import Book from "./Book";

const BookList = () => {
  useEffect(() => {
    console.log(cart); // Debugging purposes
  });

  const [books] = useState([
    { name: "Pride and Prejudice", price: "10" },
    { name: "1984", price: "15" },
    { name: "Crime and Punishment", price: "20" },
    { name: "Hamlet", price: "20" },
  ]);

  const [cart, setCart] = useState([]);

  const addToCart = (name, price) => {
    setCart((prevCart) => [...prevCart, { name, price }]);
  };

  return (
    <>
      {books.map((book, idx) => (
        <Book
          key={idx}
          name={book.name}
          price={book.price}
          addToCart={() => addToCart(book.name, book.price)}
        />
      ))}
    </>
  );
};

export default BookList;
```

**What is happening?**

1. Declaring two **states** - `books` which is an array of objects and `cart` which is an array.
2. Declaring a function called `addToCart` which adds a new object to the `cart` state.
3. Mapping through the `books` state and rendering the `Book` component. The `Book` component is given the properties - `key`, `name`, `price` and `addToCart`.

### src/components/Cart.js

In the `components` directory, create a new file called `Cart.js`. In the `Cart.js` file, add the following code:

```jsx
const Cart = () => {
  return <h3>Cart: 0</h3>;
};

export default Cart;
```

### src/App.js

In the `App.js` file, replace the existing code with the following code:

```jsx
import BookList from "./components/BookList";
import Cart from "./components/Cart";

const App = () => {
  return (
    <>
      <Cart />
      <BookList />
    </>
  );
};

export default App;
```

Run the application by running the following command:

```bash
npm run start
```

Navigate to <http://localhost:3000>. You should see following:

The screenshot below is an example the book list and an empty cart.

![](../../resources/img/10-context-api/10-context-api-1.jpeg)

The screenshot below is an example the book list and a cart with two books when the **Add to cart** button is clicked twice.

![](../../resources/img/10-context-api/10-context-api-2.jpeg)

## Overview

Thus far, you have used **props** to pass data from a higher-level component to a lower-level component. **Context API** is a light-weight state management system that provides you a way to pass data from a higher-level component to a lower-level component without having to use **props**.

In a typical React data flow, components communicate with each other using **props**. For example, the parent component, i.e., `BookList` passes data, i.e., `name`, `price` and `addToCart()` to the child component, i.e., `Book`. Though this is a simple example, usually when a lower-level component that is nested several levels in the component tree needs to access data from a higher-level component, it is a common practice to pass the data down to each component in the tree until the lower-level component receives the data. In most cases, the components between the high-level component and the lower-level component do not care about that data. However, by the time the lower-level component receives the data, you may have passed the data to three or four components. Just imagine five to ten components. This is called **prop-drilling**.

While this is perfectly fine, **prop-drilling** can be difficult to manage. There are couple of commonly used third-party libraries, i.e., **Redux**, **MobX** and **Recoil** that help you avoid **prop-drilling** as well as manage **state** in your application. These libraries can be an overkill for small applications, but a good solution for medium-large applications. For small applications, I suggest **Context API** or alternatively, **react-query**.

### Refactoring

In the `root/src` directory of the `book-shop` application, create a new directory called `contexts`. In the `contexts` directory, create a new file called `CartContext.js`. In the `CartContext.js` file, add the following code:

```jsx
import { createContext, useState } from "react";

/**
 * Creates a Context object. When a component that subscribes to
 * this Context object is rendered, it will read the current context
 * value from the closest Provider, i.e., CartProvider
 */
const CartContext = createContext();

/**
 * Every Context object comes with a Provider. It allows
 * consuming components to subscribe to context changes
 *
 * The Provider accepts one prop, i.e., children. This prop is passed to
 * consuming components that are descendants of the Provider
 *
 * All consuming components that are descendants of the Provider
 * will re-render whenever the Provider's value prop, i.e., cart
 * and addToCart changes
 */
const CartProvider = (props) => {
  const [cart, setCart] = useState([]);

  const addToCart = (name, price) => {
    setCart((prevState) => [...prevState, { name, price }]);
  };

  return (
    <CartContext.Provider value={{ cart, addToCart }}>
      {props.children}
    </CartContext.Provider>
  );
};

export { CartContext, CartProvider };
```

In the `App.js` file, wrap the `Cart` and `BookList` in the `CartProvider`.

```jsx
import BookList from "./components/BookList";
import Cart from "./components/Cart";
import { CartProvider } from "./contexts/CartContext";

const App = () => {
  return (
    <>
      <CartProvider>
        <Cart />
        <BookList />
      </CartProvider>
    </>
  );
};

export default App;
```

In the following components, import the `useContext` hook and `CartContext`. The `useContext` hook accepts a context object, i.e., `CartContext` and returns the current context value for that context.

```jsx
import { useContext } from "react";
import { CartContext } from "../contexts/CartContext";

const Book = (props) => {
  const { addToCart } = useContext(CartContext);

  return (
    <>
      <h1>{props.name}</h1>
      <h3>${props.price}</h3>
      <button onClick={() => addToCart(props.name, props.price)}>
        Add to cart
      </button>
    </>
  );
};

export default Book;
```

```jsx
import { useContext, useEffect, useState } from "react";
import Book from "./Book";
import { CartContext } from "../contexts/CartContext";

const BookList = () => {
  const { cart } = useContext(CartContext);

  useEffect(() => {
    console.log(cart); // Debugging purposes
  });

  const [books] = useState([
    { name: "Pride and Prejudice", price: "10" },
    { name: "1984", price: "15" },
    { name: "Crime and Punishment", price: "20" },
    { name: "Hamlet", price: "20" },
  ]);

  return (
    <>
      {books.map((book, idx) => (
        <Book key={idx} name={book.name} price={book.price} />
      ))}
    </>
  );
};

export default BookList;
```

```jsx
import { useContext } from "react";
import { CartContext } from "../contexts/CartContext";

const Cart = () => {
  const { cart } = useContext(CartContext);
  return (
    <>
      <h3>Cart: {cart.length}</h3>
    </>
  );
};

export default Cart;
```

## Formative Assessment

### Task Tahi

If you have not already, implement the code examples above before you move on to **Task Rua**.

### Task Rua

In the `components` directory, create a new file called `Checkout.js`. In the `Checkout.js` file, display all books in the `cart`. Display as you wish, i.e., in a list or table. Once you have completed **task rua**, move on to **Task Toru**.

### Task Toru

Using **React Router**, create new route that will render the `Checkout` component. Test the changes before you move on to the **Code Review**.

**Expected output:**

The screenshot below is an example of the book list and an empty cart. **Note:** the cart is clickable and when clicked, it will navigate you to <http://localhost:3000/checkout>.

![](../../resources/img/10-context-api/10-context-api-3.jpeg)

The screenshot below is an example of the book list and a cart with three books.

![](../../resources/img/10-context-api/10-context-api-4.jpeg)

The screenshot below is an example of the checkout cart.

![](../../resources/img/10-context-api/10-context-api-5.jpeg)

---

## Code Review

Once you have completed all tasks, open a pull request and assign **grayson-orr** as a reviewer. Please do not merge the pull request.

# Javascript-Day-12
18h. In data/cart.js, create an async function loadCartFetch() and create an async await version of loadCart(). console.log() the text attached to the response. In scripts/checkout.js, inside loadPage(), replace loadCart() with loadCartFetch().
```
export async function loadCartFetch() {
  const response = await fetch('https://supersimplebackend.dev/cart');
  const text = await response.text();
  console.log(text);
  return text;
}
```
18i. In checkout.js, use Promise.all to run loadProductsFetch() and loadCartFetch() at the same time. Note: give the promises directly to Promise.all (don't await them, otherwise they will run one at a time). Then, use await Promise.all(...) to wait for Promise.all to finish.
```
await Promise.all([
  loadProductsFetch(),
  loadCartFetch()
]);
```
18j. In orderSummaryTest.js, in the beforeAll hook, instead of using a done function, make the inner function async and use await to wait for loadProductsFetch() to finish.
```
 beforeAll(async () => {
    await loadProductsFetch();
  });
```
181. We'll finish the orders page. Create a new file scripts/orders.js for creating the orders page, and load it in orders.html. Using the array of orders in data/orders.js, generate the HTML for this page.
```
<div class="orders-grid js-orders-grid">

<script type="module" src="scripts/orders.js"></script>

import {getProduct, loadProductsFetch} from '../data/products.js';
import {orders} from '../data/orders.js';
import dayjs from 'https://unpkg.com/dayjs@1.11.10/esm/index.js';
import formatCurrency from './utils/money.js';

async function loadPage() {
  await loadProductsFetch();

  let ordersHTML = '';

  orders.forEach((order) => {
    const orderTimeString = dayjs(order.orderTime).format('MMMM D');

    ordersHTML += `
      <div class="order-container">
        <div class="order-header">
          <div class="order-header-left-section">
            <div class="order-date">
              <div class="order-header-label">Order Placed:</div>
              <div>${orderTimeString}</div>
            </div>
            <div class="order-total">
              <div class="order-header-label">Total:</div>
              <div>$${formatCurrency(order.totalCostCents)}</div>
            </div>
          </div>
          <div class="order-header-right-section">
            <div class="order-header-label">Order ID:</div>
            <div>${order.id}</div>
          </div>
        </div>
        <div class="order-details-grid">
          ${productsListHTML(order)}
        </div>
      </div>
    `;
  });

  function productsListHTML(order) {
    let productsListHTML = '';

    order.products.forEach((productDetails) => {
      const product = getProduct(productDetails.productId);

      productsListHTML += `
        <div class="product-image-container">
          <img src="${product.image}">
        </div>
        <div class="product-details">
          <div class="product-name">
            ${product.name}
          </div>
          <div class="product-delivery-date">
            Arriving on: ${
              dayjs(productDetails.estimatedDeliveryTime).format('MMMM D')
            }
          </div>
          <div class="product-quantity">
            Quantity: ${productDetails.quantity}
          </div>
          <button class="buy-again-button button-primary">
            <img class="buy-again-icon" src="images/icons/buy-again.png">
            <span class="buy-again-message">Buy it again</span>
          </button>
        </div>
        <div class="product-actions">
          <a href="tracking.html?orderId=${order.id}&productId=${product.id}">
            <button class="track-package-button button-secondary">
              Track package
            </button>
          </a>
        </div>
      `;
    });

    return productsListHTML;
  }

  document.querySelector('.js-orders-grid').innerHTML = ordersHTML;
```
18m. Make the orders page interactive:
"Buy it again" button should add the product to the cart.
"Track package" button should open the tracking page (remember to insert the orderld and productid into the URL parameters).
```
import {addToCart} from '../data/cart.js';

<button class="buy-again-button button-primary js-buy-again"
    data-product-id="${product.id}">

 document.querySelectorAll('.js-buy-again').forEach((button) => {
    button.addEventListener('click', () => {
      addToCart(button.dataset.productId);

      // (Optional) display a message that the product was added,
      // then change it back after a second.
      button.innerHTML = 'Added';
      setTimeout(() => {
        button.innerHTML = `
          <img class="buy-again-icon" src="images/icons/buy-again.png">
          <span class="buy-again-message">Buy it again</span>
        `;
      }, 1000);
    });
  });
```
18n. We'll finish the tracking page. Create a new file scripts/tracking.js and load it in tracking.html. Then, go to the orders page, and click "Track package" to go to the tracking page. This will save the orderId and productId in the URL parameters (make the URL always has these 2 parameters, otherwise the tracking page won't work).
Use orderId and productId from the URL parameters to get the order and product to track. Use this data to generate the HTML for this page.
```
export function getOrder(orderId) {
  let matchingOrder;

  orders.forEach((order) => {
    if (order.id === orderId) {
      matchingOrder = order;
    }
  });

  return matchingOrder;
}

import {getOrder} from '../data/orders.js';
import {getProduct, loadProductsFetch} from '../data/products.js';
import dayjs from 'https://unpkg.com/dayjs@1.11.10/esm/index.js';

async function loadPage() {
  await loadProductsFetch();

  const url = new URL(window.location.href);
  const orderId = url.searchParams.get('orderId');
  const productId = url.searchParams.get('productId');

  const order = getOrder(orderId);
  const product = getProduct(productId);

  // Get additional details about the product like
  // the estimated delivery time.
  let productDetails;
  order.products.forEach((details) => {
    if (details.productId === product.id) {
      productDetails = details;
    }
  });

  const trackingHTML = `
    <a class="back-to-orders-link link-primary" href="orders.html">
      View all orders
    </a>
    <div class="delivery-date">
      Arriving on ${
        dayjs(productDetails.estimatedDeliveryTime).format('dddd, MMMM D')
      }
    </div>
    <div class="product-info">
      ${product.name}
    </div>
    <div class="product-info">
      Quantity: ${productDetails.quantity}
    </div>
    <img class="product-image" src="${product.image}">
    <div class="progress-labels-container">
      <div class="progress-label">
        Preparing
      </div>
      <div class="progress-label current-status">
        Shipped
      </div>
      <div class="progress-label">
        Delivered
      </div>
    </div>
    <div class="progress-bar-container">
      <div class="progress-bar"></div>
    </div>
  `;

  document.querySelector('.js-order-tracking').innerHTML = trackingHTML;
}

loadPage();
```
180. Make the tracking page interactive:
• Calculate the percent progress of the delivery: ((currentTime - orderTime) / (deliveryTime - orderTime)) * 100
Set the width of the green progress bar to this percent. Hint: add the style attribute: style="width: _%;"
Set the correct status above the progress bar to green (0% -49% = Preparing, 50%-99% = Shipped, 100+% = Delivered).
```
 const today = dayjs();
  const orderTime = dayjs(order.orderTime);
  const deliveryTime = dayjs(productDetails.estimatedDeliveryTime);
  const percentProgress = ((today - orderTime) / (deliveryTime - orderTime)) * 100;

<div class="progress-label ${
        percentProgress < 50 ? 'current-status' : ''
      }">

<div class="progress-label ${
        (percentProgress >= 50 && percentProgress < 100) ? 'current-status' : ''
      }">

<div class="progress-label ${
        percentProgress >= 100 ? "current-status" : ''
      }">
<div class="progress-bar" style="width: ${percentProgress}%;"></div>

```
18p. At the top of the home page (amazon.html) there's a search bar:
When you type in the search bar and press the search button, it should go to the home page (amazon.html) and also save your search in a URL parameter: amazon.html?search=your search
• On the home page, check if there's a URL parameter called search. If it exists, filter the products on the home page and only show products whose name contains what you searched (hint: use.includes()).
```
<input class="search-bar js-search-bar" type="text" placeholder="Search">
    <button class="search-button js-search-button">

const url = new URL(window.location.href);
  const search = url.searchParams.get('search');

  let filteredProducts = products;
  if (search) {
    filteredProducts = products.filter((product) => {
      return product.name.includes(search);
    });
  }

document.querySelector('.js-search-button')
    .addEventListener('click', () => {
      const search = document.querySelector('.js-search-bar').value;
      window.location.href = `amazon.html?search=${search}`;
    });
}
```
18q. We'll improve the search feature from 18p:
Make the search case-insensitive (capitals don't matter). Each product has a property called "keywords". Add this property to the Product class. When filtering the products, also check if one of the keywords contains what you searched (case-insensitive).
```
let matchingKeyword = false;

  product.keywords.forEach((keyword) => {
    if (keyword.toLowerCase().includes(search.toLowerCase())) {
      matchingKeyword = true;
    }
  });

  return matchingKeyword ||
    product.name.toLowerCase().includes(search.toLowerCase());
});
```

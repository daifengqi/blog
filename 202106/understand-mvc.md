# 通过一个小项目理解MVC理念

# Understand MVC Via a Mini Project

## 2021 年 6 月 24 日

## Jun 24 2021



### 前言

这是我在上的区块链课程的期末project，是做一个关于天气的“智能合约”，大概思想是，合约的双方对未来的天气进行一次赌博，比如打赌某天会不会下雨，某个时辰的温度是否大于某个特定的度数（strike），然后这个合约会被放到以太坊的链上，到时间合约代码会自动调取天气的API来判断谁获胜，那么获胜的一方就会赢的两人预先抵押的以太币作为回报。

思想很简洁，实现起来需要涉及到前后端和数据库，所以算是一个小全栈项目，下面通过这个项目写一下我对MVC架构的理解（由于project需要用英文提交，我这里直接放英文结果了）。



### Programming Implementation

**Steps of Implementing a Smart Contract**

Find the API of the test network (Ropsten), Ethereum wallet (Metamask) and web3.js, and complete the following tasks by sending get / post request:

- Step 1. Account registration in Ethereum wallet.
- Step 2. Get ETH from the test network.
- Step 3. Release smart contract.

This part will illustrate some details on how to implement a entity to let us really operate on such products. The expected presentation form could be Web application, desktop software, mobile app or Wechat Mini Program.

Let's introduce the part from bottom to top, under a software design pattern Model– view–controller (usually known as MVC).

### Database: The Part of Model

Model is the central component of the pattern. It is about the application's data structure, independent of the user interface. It directly manages the data, logic and rules of the application. We need to define the database scheme before we initializing the app.

For some particular reason, we choose to use a NoSQL database, MongoDB, to store all the data. MongoDB is a document-oriented database which means that it holds JSON-like documents inside the collections. The collections we will create are as follows.

- Firstly, since our smart contracts need more than one participants, we need to abstract the participants, which can be understood as the two sides of a gambling smart contract. We create a database called "User". The users must have some required keys like the user-name, the hashed-name (which is useful for anonymous trading), the password, the matched Ethereum account and its balance, and all the smart contracts that are related to the user.
- Secondly and similarly, we need a "Contract" schema to store all the smart contracts that have been published. The information includes the initiator and receiver of the contract, the strike weather description (mostly the temperature, also it can be whether one day will rain or not in the future), and so on.

### Backend: The Part of Controller

We use NodeJs which is a JavaScript runtime to execute the backend code. This part is required to connect the UI and the database. Every interaction from users on the interface may have influence on the change of data we have stored. The backend needs to change the information in the database and reflect the variation on the UI in time if necessary.

Npm (originally short for Node Package Manager) is a package manager for the JavaScript programming language. Since we need to do a lot of things, the third-party packages are necessary to help us complete the task. In the following sections we will list some important packages and briefly explain their functions.

- express: "express" is a is a backend web application framework for Node.js. it is relatively minimal with many features available as plugins such as routers, body-parsers and cross-origin resource sharing (CORS). There is also a new similar framework called "koa", but we will still insist on this.
- nodemailer: "nodemailer" can let Node.js operate on email receiving and sending jobs. We use the package to send verification code to users when they sign up the first time

Other small packages are also useful, such as "bcryptjs" that provides hash function, "jsonwebtoken" that lets us give token to users, and "axios" for based HTTP client for the browser and Node.js.

### 4.4 UI: The Part of View

We will use a frontend framework called React that is developed by a Facebook team. One of the reason we choose React is that the cross-end development is well supported under the efforts of Facebook Team and some domestic lab. We can really write once, and let the code run anywhere including Web, desktop and mobile apps, and even Wechat mini-program.

By the way, a similar framework developed by a Chinese person is called VUE (the pronunciation is the same as "view"). Something interesting is that React is more like a layer of "view" but VUE is more "reactive". They both support virtual DOM which can be understood as a primary version of HTML DOM. The operating on HTML DOM is slow and expensive, and these two frameworks solve the problem by operating on virtual DOM instead of real ones.

Basically, we have three UI pages including the weather page, the contract square page and the user page, which means we need to use router to go through these pages. This increases the difficulty of this project.

The weather information or data is from a website called "openweathermap". It provides free and public API to get weather data, but has restrictions on number of calls. It does not matter when we have few users, which is indeed the truth.


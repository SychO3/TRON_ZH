# 构建 WEB3 应用程序

本文涵盖了从零开始开发 DApp 的整个过程，从合同编写到部署和启动。
通过学习如何构建去中心化库，开发者可以轻松掌握在 TRON 网络上部署自己的去中心化应用程序的流程。

## 准备工作

### 安装

**Node v10+**

```shell
# node -v
v10.24.1
```

**TronLink**

安装 [Tronlink](https://www.tronlink.org/) 的 Chrome 浏览器扩展。

## 我们在做什么

我们正在构建一个去中心化的库，它包含以下功能：

- Book Borrowing
- Book Browsing
- Book Adding

从[此处](https://github.com/TRON-Developer-Hub/decentralized-library)下载完整的项目代码，然后运行 `npm install` 安装依赖项。

## 数据结构

通常情况下，借阅者关心的是图书的标题、内容、可用性和价格。
在此基础上，我们在合同中设计了一个名为 "图书 "的结构，它包含以下属性：

```solidity
struct Book {
       string name;
       string description;
       bool valid;      // false if been borrowed
       uint256 price;   // TRX per day
       address owner;   // owner of the book
}
```

我们希望图书馆能以一种简便的方式找到每本书。
为此，我们建立了一个 `bookId` 属性和 `bookId` 与 `Book` 之间的映射关系，命名为 `books`。

```solidity
uint256 public bookId;

mapping (uint256 => Book) public books;
```

此外，我们还必须记录每本书的租借信息，包括借书人、开始和结束日期。

与 Book 一样，构建一个名为 Tracking 的结构来跟踪这些数据。该结构体拥有以下字段：

```solidity
struct Tracking {
       uint256 bookId;
       uint256 startTime; // start time, in timestamp
       uint256 endTime; // end time, in timestamp
       address borrower; // borrower's address
}
```

同样，必须建立映射关系来管理每个租赁记录：

```solidity
uint256 public trackingId;

mapping(uint256 => Tracking) public trackings;
```

## 功能和事件

我们正在为图书馆添加基本功能，包括：

- 添加图书 - `addBook`
- 借书 - `borrowBook`
- 删除图书 - `deleteBook`

### addBook

```solidity
/**
* @dev Add a Book with predefined `name`, `description` and `price`
* to the library.
*
* Returns a boolean value indicating whether the operation succeeded.
*
* Emits a {NewBook} event.
*/
function addBook(string memory name, string memory description, uint256 price) public returns (bool) {
       Book memory book = Book(name, description, true, price, _msgSender());

       books[bookId] = book;

       emit NewBook(bookId++);

       return true;
}

/**
* @dev Emitted when a new book is added to the library.
* Note bookId starts from 0.
*/
event NewBook(uint256 bookId);
```

### borrowBook

```solidity
/**
* @dev Borrow a book has `_bookId`. The rental period starts from
* `startTime` ends with `endTime`.
*
* Returns a boolean value indicating whether the operation succeeded.
*
* Emits a `NewRental` event.
*/
function borrowBook(uint256 _bookId, uint256 startTime, uint256 endTime) public payable returns(bool) {
       Book storage book = books[_bookId];

       require(book.valid == true, "The book is currently on loan");

       require(_msgValue() == book.price * _days(startTime, endTime), "Incorrect fund sent.");

       _sendTRX(book.owner, _msgValue());

       _createTracking(_bookId, startTime, endTime);

       emit NewRental(_bookId, trackingId++);
}
```

### deleteBook

```solidity
/**
* @dev Delete a book from the library. Only the book's owner or the
* library's owner is authorised for this operation.
*
* Returns a boolean value indicating whether the operation succeeded.
*
* Emits a `DeleteBook` event.
*/
function deleteBook(uint256 _bookId) public returns(bool) {
       require(_msgSender() == books[_bookId].owner || isOwner(),
               "You are not authorised to delete this book.");
      
       delete books[_bookId];

       emit DeleteBook(_bookId);

       return true;
}
```

### _sendTRX

```solidity
/**
* @dev Send TRX to the book's owner.
*/
function _sendTRX(address receiver, uint256 value) internal {
       payable(address(uint160(receiver))).transfer(value);
}
```

### _createTracking

```solidity
/**
* @dev Create a new rental tracking.
*/
function _createTracking(uint256 _bookId, uint256 startTime, uint256 endTime) internal {
         trackings[trackingId] = Tracking(_bookId, startTime, endTime, _msgSender());

         Book storage book = books[_bookId];

         book.valid = false;
 }
```

合同已经完成，是时候部署了。

## 部署和测试

**更多内容等待翻译完成。。。**


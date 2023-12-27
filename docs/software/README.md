# Реалізація інформаційного та програмного забезпечення

В рамках проекту розробляється: 
- SQL-скрипт для створення на початкового наповнення бази даних
```sql
-- MySQL Workbench Forward Engineering

SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';

-- -----------------------------------------------------
-- Schema mydb
-- -----------------------------------------------------

-- -----------------------------------------------------
-- Schema mydb
-- -----------------------------------------------------
CREATE SCHEMA IF NOT EXISTS `mydb` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci ;
USE `mydb` ;

-- -----------------------------------------------------
-- Table `mydb`.`content`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `mydb`.`content` (
  `idContent` INT NOT NULL AUTO_INCREMENT,
  `title` VARCHAR(45) NOT NULL,
  `description` VARCHAR(45) NOT NULL,
  `status` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`idContent`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COLLATE = utf8mb4_0900_ai_ci;


-- -----------------------------------------------------
-- Table `mydb`.`contentanalysistask`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `mydb`.`contentanalysistask` (
  `idContentAnalysisTask` INT NOT NULL AUTO_INCREMENT,
  `title` VARCHAR(45) NOT NULL,
  `deadline` DATETIME NOT NULL,
  `analyst` VARCHAR(45) NOT NULL,
  `ContentId` INT NOT NULL,
  PRIMARY KEY (`idContentAnalysisTask`),
  INDEX `fk_ContentAnalysisTask_Content1_idx` (`ContentId` ASC) ,
  CONSTRAINT `fk_ContentAnalysisTask_Content1`
    FOREIGN KEY (`ContentId`)
    REFERENCES `mydb`.`content` (`idContent`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COLLATE = utf8mb4_0900_ai_ci;


-- -----------------------------------------------------
-- Table `mydb`.`user`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `mydb`.`user` (
  `idUser` INT NOT NULL AUTO_INCREMENT,
  `login` VARCHAR(45) NOT NULL,
  `email` VARCHAR(45) NOT NULL,
  `role` VARCHAR(45) NOT NULL,
  `password` VARCHAR(45) NOT NULL,
  `picture` MEDIUMBLOB NULL DEFAULT NULL,
  PRIMARY KEY (`idUser`))
ENGINE = InnoDB
AUTO_INCREMENT = 6
DEFAULT CHARACTER SET = utf8mb4
COLLATE = utf8mb4_0900_ai_ci;


-- -----------------------------------------------------
-- Table `mydb`.`member`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `mydb`.`member` (
  `idMember` INT NOT NULL AUTO_INCREMENT,
  `UserId` INT NOT NULL,
  `ContentId` INT NOT NULL,
  PRIMARY KEY (`idMember`),
  INDEX `fk_Member_User_idx` (`UserId` ASC),
  INDEX `fk_Member_Content1_idx` (`ContentId` ASC),
  CONSTRAINT `fk_Member_Content1`
    FOREIGN KEY (`ContentId`)
    REFERENCES `mydb`.`content` (`idContent`),
  CONSTRAINT `fk_Member_User`
    FOREIGN KEY (`UserId`)
    REFERENCES `mydb`.`user` (`idUser`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COLLATE = utf8mb4_0900_ai_ci;


-- -----------------------------------------------------
-- Table `mydb`.`paymentdata`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `mydb`.`paymentdata` (
  `idPaymentData` INT NOT NULL AUTO_INCREMENT,
  `cardNumber` INT NOT NULL,
  `cardExpireDate` DATETIME NOT NULL,
  `cardCVV` INT NOT NULL,
  `email` VARCHAR(45) NOT NULL,
  `ContentId` INT NOT NULL,
  PRIMARY KEY (`idPaymentData`),
  INDEX `fk_PaymentData_Content1_idx` (`ContentId` ASC),
  CONSTRAINT `fk_PaymentData_Content1`
    FOREIGN KEY (`ContentId`)
    REFERENCES `mydb`.`content` (`idContent`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COLLATE = utf8mb4_0900_ai_ci;


-- -----------------------------------------------------
-- Table `mydb`.`review`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `mydb`.`review` (
  `idReview` INT NOT NULL AUTO_INCREMENT,
  `text` VARCHAR(45) NOT NULL,
  `rate` INT NOT NULL,
  `ContentId` INT NOT NULL,
  PRIMARY KEY (`idReview`),
  INDEX `fk_Review_Content1_idx` (`ContentId` ASC),
  CONSTRAINT `fk_Review_Content1`
    FOREIGN KEY (`ContentId`)
    REFERENCES `mydb`.`content` (`idContent`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COLLATE = utf8mb4_0900_ai_ci;

INSERT INTO `user` VALUES (1,'Alex','alexe@gmail.com','admin','142569',NULL),
(2,'Stas','stas@gmail.com','user','123789',NULL),
(3,'Farid','farid@gmail.com','user','741963',NULL),
(4,'Anastasia','Anastasia@gmail.com','user','145236',NULL),
(5,'Danya','danya@gmail.com','user','478569',NULL);

SET SQL_MODE=@OLD_SQL_MODE;
SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS;
SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS;
```
# RESTfull сервіс для управління даними

## Вхідний файл програми
```
const express = require("express");
const cors = require("cors");
const router = require("./routes");
const AppError = require("./utils/appError");
const errorHandler = require("./utils/errorHandler");

const app = express();
app.use(express.json());
app.use(cors());
app.use("/api", router);

app.all("*", (req, res, next) => {
  next(new AppError(`The URL ${req.originalUrl} does not exists`, 404));
});
app.use(errorHandler);

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`server running on port ${PORT}`);
});

module.exports = app;
```
## Файл для встановлення доступу до бази даних

```
const mysql = require("mysql2");
const conn = mysql.createConnection({
  host: "localhost",
  user: "root",
  password: "159357",
  database: "mydb",
});

conn.connect();

module.exports = conn;
```
# CRUD для користувачів

## Маршрути

```
const express = require("express");
const controllers = require("../controllers");
const router = express.Router();

router.route("/User").get(controllers.getAllUsers).post(controllers.createUser);
router
  .route("/User/:id")
  .get(controllers.getUserById)
  .put(controllers.updateUser)
  .delete(controllers.deleteUser);
module.exports = router;
```
## Контроллери

```
const AppError = require("../utils/appError");
const conn = require("../services/db");

exports.getAllUsers = (req, res, next) => {
  conn.query("SELECT * FROM User", function (err, data, fields) {
    if (err) return next(new AppError(err));
    res.status(200).json({
      status: "success",
      length: data?.length,
      data: data,
    });
  });
};

exports.createUser = (req, res, next) => {
  if (!req.body) return next(new AppError("No form data found", 404));
  const values = [
    req.body.username,
    req.body.email,
    req.body.password,
    req.body.Role,
  ];
  conn.query(
    "INSERT INTO User (username, email, password, Role) VALUES(?)",
    [values],
    function (err, data, fields) {
      if (err) return next(new AppError(err, 500));
      res.status(201).json({
        status: "success",
        message: "user added!",
      });
    }
  );
};

exports.getUserById = (req, res, next) => {
  if (!req.params.id) {
    return next(new AppError("No user id found", 404));
  }
  conn.query(
    "SELECT * FROM User WHERE id = ?",
    [req.params.id],
    function (err, data, fields) {
      if (err) return next(new AppError(err, 500));
      res.status(200).json({
        status: "success",
        length: data?.length,
        data: data,
      });
    }
  );
};

exports.updateUser = (req, res, next) => {
  if (!req.params.id) {
    return next(new AppError("No user id found", 404));
  }
  conn.query(
    "UPDATE User SET username=?, email=?, password=?, Role=? WHERE id=?",
    [
      req.body.username,
      req.body.email,
      req.body.password,
      req.body.Role,
      req.params.id,
    ],
    function (err, data, fields) {
      if (err) return next(new AppError(err, 500));
      res.status(201).json({
        status: "success",
        message: "user info updated!",
      });
    }
  );
};

exports.deleteUser = (req, res, next) => {
  if (!req.params.id) {
    return next(new AppError("No todo id found", 404));
  }
  conn.query(
    "DELETE FROM User WHERE id=?",
    [req.params.id],
    function (err, fields) {
      if (err) return next(new AppError(err, 500));
      res.status(201).json({
        status: "success",
        message: "user deleted!",
      });
    }
  );
};
```
## Обробники помилок

```
class AppError extends Error {
    constructor(msg, statusCode) {
      super(msg);
  
      this.statusCode = statusCode;
      this.error = `${statusCode}`.startsWith("4") ? "fail" : "error";
      this.isOperational = true;
  
      Error.captureStackTrace(this, this.constructor);
    }
  }
module.exports = AppError;
```
```
module.exports = (err, req, res, next) => {
    err.statusCode = err.statusCode || 500;
    err.status = err.status || "error";
    res.status(err.statusCode).json({
      status: err.status,
      message: err.message,
    });
  };
```


# E-commerce Selenium BDD Project — Ready-to-upload files

> This document contains the full project files for **Project 1: E‑Commerce Web App Testing** (Selenium + Cucumber BDD + Java + Maven). The project uses **Chrome only** (as requested) via WebDriverManager.

---

## Project structure

```
E-commerce-Sample-Automation-Project/
├── pom.xml
├── README.md
├── src
│   ├── main
│   │   └── java
│   │       └── com.mithil.framework
│   │           ├── pages
│   │           │   ├── HomePage.java
│   │           │   ├── ProductPage.java
│   │           │   ├── CartPage.java
│   │           │   └── CheckoutPage.java
│   │           └── utils
│   │               └── DriverFactory.java
│   └── test
│       └── java
│           └── com.mithil.tests
│               ├── runners
│               │   └── TestRunner.java
│               ├── stepdefinitions
│               │   ├── HomeSteps.java
│               │   └── CheckoutSteps.java
│               └── hooks
│                   └── Hooks.java
└── src/test/resources
    └── features
        └── ecommerce.feature
```

---

> **Note:** All files below are shown as copy-pasteable content. Put them in the paths shown above.

---

## 1) `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mithil</groupId>
    <artifactId>ecommerce-automation</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <cucumber.version>7.11.2</cucumber.version>
    </properties>

    <dependencies>
        <!-- Selenium -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.11.0</version>
        </dependency>

        <!-- Cucumber -->
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <version>${cucumber.version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit</artifactId>
            <version>${cucumber.version}</version>
        </dependency>

        <!-- JUnit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>

        <!-- WebDriverManager (Chrome driver management) -->
        <dependency>
            <groupId>io.github.bonigarcia</groupId>
            <artifactId>webdrivermanager</artifactId>
            <version>5.4.1</version>
        </dependency>

        <!-- ExtentReports for a simple report (optional) -->
        <dependency>
            <groupId>com.aventstack</groupId>
            <artifactId>extentreports</artifactId>
            <version>5.0.9</version>
        </dependency>

        <!-- Gson (optional, for utils) -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.10.1</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
                <configuration>
                    <includes>
                        <include>**/TestRunner.java</include>
                    </includes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 2) `DriverFactory.java` (src/main/java/com/mithil/framework/utils)

```java
package com.mithil.framework.utils;

import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

public class DriverFactory {
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() {
        if (driver.get() == null) {
            setupDriver();
        }
        return driver.get();
    }

    private static void setupDriver() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        // run tests in headless mode if you want to speed up CI - commented by default
        // options.addArguments("--headless=new");
        options.addArguments("--start-maximized");
        options.addArguments("--disable-notifications");
        driver.set(new ChromeDriver(options));
    }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```

---

## 3) Page Objects (src/main/java/com/mithil/framework/pages)

### `HomePage.java`

```java
package com.mithil.framework.pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

public class HomePage {
    private WebDriver driver;

    private By searchBox = By.id("small-searchterms");
    private By searchButton = By.cssSelector("button[type='submit'][class*='search-box-button']");
    private By firstProduct = By.cssSelector(".product-item .product-title a");

    public HomePage(WebDriver driver) {
        this.driver = driver;
    }

    public void openHomePage() {
        driver.get("https://demo.nopcommerce.com/");
    }

    public void search(String keyword) {
        WebElement input = driver.findElement(searchBox);
        input.clear();
        input.sendKeys(keyword);
        driver.findElement(searchButton).click();
    }

    public void clickFirstProduct() {
        driver.findElement(firstProduct).click();
    }
}
```

### `ProductPage.java`

```java
package com.mithil.framework.pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class ProductPage {
    private WebDriver driver;
    private By addToCartBtn = By.id("add-to-cart-button-4"); // note: id may vary per product; adapt if necessary
    private By shoppingCartLink = By.cssSelector("a[href='/cart']");

    public ProductPage(WebDriver driver) {
        this.driver = driver;
    }

    public void addToCart() {
        driver.findElement(addToCartBtn).click();
        // small wait to let cart update (replace with explicit wait in production)
        try { Thread.sleep(1500); } catch (InterruptedException e) { /* ignore */ }
    }

    public void goToCart() {
        driver.findElement(shoppingCartLink).click();
    }
}
```

### `CartPage.java`

```java
package com.mithil.framework.pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class CartPage {
    private WebDriver driver;
    private By termsCheckbox = By.id("termsofservice");
    private By checkoutButton = By.id("checkout");

    public CartPage(WebDriver driver) {
        this.driver = driver;
    }

    public void acceptTerms() {
        driver.findElement(termsCheckbox).click();
    }

    public void proceedToCheckout() {
        driver.findElement(checkoutButton).click();
    }
}
```

### `CheckoutPage.java`

```java
package com.mithil.framework.pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class CheckoutPage {
    private WebDriver driver;

    // Because nopCommerce has many form fields and flow, we'll keep this minimal for demo
    private By continueAsGuestBtn = By.cssSelector("button.button-1.checkout-as-guest-button");
    private By billingContinue = By.cssSelector("button[name='save'][class*='billing-address-next-step-button']");
    private By shippingContinue = By.cssSelector("button[class*='shipping-method-next-step-button']");
    private By paymentContinue = By.cssSelector("button[class*='payment-method-next-step-button']");
    private By confirmOrderBtn = By.cssSelector("button[class*='confirm-order-next-step-button']");
    private By orderSuccessMsg = By.cssSelector(".section.order-completed .title");

    public CheckoutPage(WebDriver driver) {
        this.driver = driver;
    }

    public void continueAsGuest() {
        if (!driver.findElements(continueAsGuestBtn).isEmpty()) {
            driver.findElement(continueAsGuestBtn).click();
        }
    }

    public void clickBillingContinue() {
        if (!driver.findElements(billingContinue).isEmpty()) {
            driver.findElement(billingContinue).click();
        }
    }

    public void clickShippingContinue() {
        if (!driver.findElements(shippingContinue).isEmpty()) {
            driver.findElement(shippingContinue).click();
        }
    }

    public void clickPaymentContinue() {
        if (!driver.findElements(paymentContinue).isEmpty()) {
            driver.findElement(paymentContinue).click();
        }
    }

    public void confirmOrder() {
        if (!driver.findElements(confirmOrderBtn).isEmpty()) {
            driver.findElement(confirmOrderBtn).click();
        }
    }

    public String getOrderSuccessText() {
        if (!driver.findElements(orderSuccessMsg).isEmpty()) {
            return driver.findElement(orderSuccessMsg).getText();
        }
        return "";
    }
}
```

---

## 4) Feature file (src/test/resources/features/ecommerce.feature)

```gherkin
Feature: E-commerce search and checkout
  As a shopper
  I want to search for a product, add it to cart and complete checkout

  Scenario: Search for a T-shirt and complete a guest checkout
    Given the user opens the home page
    When the user searches for "t-shirt"
    And the user selects the first product
    And the user adds the product to the cart
    And the user goes to the cart and accepts terms
    And the user proceeds to checkout
    Then the order completion page should be displayed
```

---

## 5) Step Definitions (src/test/java/com/mithil/tests/stepdefinitions)

### `HomeSteps.java`

```java
package com.mithil.tests.stepdefinitions;

import com.mithil.framework.pages.CartPage;
import com.mithil.framework.pages.HomePage;
import com.mithil.framework.pages.ProductPage;
import com.mithil.framework.utils.DriverFactory;
import io.cucumber.java.en.And;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.When;
import org.openqa.selenium.WebDriver;

public class HomeSteps {
    private WebDriver driver = DriverFactory.getDriver(
```

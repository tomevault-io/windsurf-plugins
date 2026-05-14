---
trigger: always_on
description: Cursor rules for GitHub development with instructions integration.
---

Writing code is like giving a speech. If you use too many big words, you confuse your audience. Define every word, and you end up putting your audience to sleep. Similarly, when you write code, you shouldn't just focus on making it work. You should also aim to make it readable, understandable, and maintainable for future readers. To paraphrase software engineer Martin Fowler, "Anybody can write code that a computer can understand. Good programmers write code that humans can understand."

As software developers, understanding how to write clean code that is functional, easy to read, and adheres to best practices helps you create better software consistently.

This article discusses what clean code is and why it's essential and provides principles and best practices for writing clean and maintainable code.

What Is Clean Code?

Clean code is a term used to refer to code that is easy to read, understand, and maintain. It was made popular by Robert Cecil Martin, also known as Uncle Bob, who wrote "Clean Code: A Handbook of Agile Software Craftsmanship" in 2008. In this book, he presented a set of principles and best practices for writing clean code, such as using meaningful names, short functions, clear comments, and consistent formatting.

Ultimately, the goal of clean code is to create software that is not only functional but also readable, maintainable, and efficient throughout its lifecycle.

Why Is Clean Code Important?

When teams adhere to clean code principles, the code base is easier to read and navigate, which makes it faster for developers to get up to speed and start contributing. Here are some reasons why clean code is essential.

Readability and maintenance: Clean code prioritizes clarity, which makes reading, understanding, and modifying code easier. Writing readable code reduces the time required to grasp the code's functionality, leading to faster development times.

Team collaboration: Clear and consistent code facilitates communication and cooperation among team members. By adhering to established coding standards and writing readable code, developers easily understand each other's work and collaborate more effectively.

Debugging and issue resolution: Clean code is designed with clarity and simplicity, making it easier to locate and understand specific sections of the codebase. Clear structure, meaningful variable names, and well-defined functions make it easier to identify and resolve issues.

Improved quality and reliability: Clean code prioritizes following established coding standards and writing well-structured code. This reduces the risk of introducing errors, leading to higher-quality and more reliable software down the line.

Now that we understand why clean code is essential, let's delve into some best practices and principles to help you write clean code.

Principles of Clean Code

Like a beautiful painting needs the right foundation and brushstrokes, well-crafted code requires adherence to specific principles. These principles help developers write code that is clear, concise, and, ultimately, a joy to work with.

Let's dive in.

1. Avoid Hard-Coded Numbers

Use named constants instead of hard-coded values. Write constants with meaningful names that convey their purpose. This improves clarity and makes it easier to modify the code.

Example:

The example below uses the hard-coded number 0.1 to represent a 10% discount. This makes it difficult to understand the meaning of the number (without a comment) and adjust the discount rate if needed in other parts of the function.

Before:

def calculate_discount(price):  
  discount = price * 0.1 # 10% discount  
  return price - discount

The improved code replaces the hard-coded number with a named constant TEN_PERCENT_DISCOUNT. The name instantly conveys the meaning of the value, making the code more self-documenting.

After:

def calculate_discount(price):  
  TEN_PERCENT_DISCOUNT = 0.1  
  discount = price * TEN_PERCENT_DISCOUNT  
  return price - discount

Also, If the discount rate needs to be changed, it only requires modifying the constant declaration, not searching for multiple instances of the hard-coded number.

2. Use Meaningful and Descriptive Names

Choose names for variables, functions, and classes that reflect their purpose and behavior. This makes the code self-documenting and easier to understand without extensive comments. As Robert Martin puts it, “A name should tell you why it exists, what it does, and how it is used. If a name requires a comment, then the name does not reveal its intent.”

Example:

If we take the code from the previous example, it uses generic names like "price" and "discount," which leaves their purpose ambiguous. Names like "price" and "discount" could be interpreted differently without context.

Before:

def calculate_discount(price):  
  TEN_PERCENT_DISCOUNT = 0.1  
  discount = price * TEN_PERCENT_DISCOUNT  
  return price - discount

Instead, you can declare the variables to be more descriptive.

After:

def calculate_discount(product_price):  
  TEN_PERCENT_DISCOUNT = 0.1  
  discount_amount = product_price * TEN_PERCENT_DISCOUNT  
  return product_price - discount_amount


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

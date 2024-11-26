# Schema Validation with Zod (medium.com)

<https://medium.com/@imartinflores/schema-validation-with-zod-9859328daf3a>

## Tags

#nodejs #zod

## Content

# Schema Validation with Zod - Ignacio Martin - Medium {#schema-validation-with-zod---ignacio-martin---medium .reader-title}

Ignacio Martin

------------------------------------------------------------------------

[](https://medium.com/@imartinflores?source=post_page---byline--9859328daf3a--------------------------------){rel="noopener follow"}

![Ignacio Martin](https://miro.medium.com/v2/resize:fill:88:88/1*WKif-thXft73DHyFGN-aIA.png){testid="authorPhoto" loading="lazy" height="44" width="44"}

As you may know, API Testing is considered one of the most important and extensive part in the Test Pyramid.\
And there are a couple of reasons for that if it's done properly.

-   [Reliability]{#a76b}
-   [Time effective]{#8572}
-   [Maintenance]{#78ba}
-   [Stability]{#aba5}

And the list goes on, but we are not going to mention all of them.

## So now, what is **"Schema Validation"**? {#52c6}

By definition, this is the process of validating your outgoing and incoming data against a predefined schema/structure.\
This helps you to make sure that the format (And the content) of your data is correct.

## And why do we need it ? {#002f}

Well, most of applications integrate with some other parts (network calls, databases..) --- This is where we need to make sure that integration is working. Having said that, this is something we can't do with Unit Tests, but also, we do not always need a UI to validate it.

## How can we do this ? {#7846}

Basically, this can be done by getting what you need from your API or Integration Service and comparing it with a Data Model.

The data model can be a *.json* file stored somewhere in your project, or directly hardcoded in your test.\
Luckily, there is a tool called **"Zod"** to help with this, and we will be going through it soon.

<figure>

</figure>

**Zod** is a JavaScript library that provides a nice and easy way of validating this data. It allows you to define "Schemas" using declarative syntax, rules, required fields, etc, to perform validations effortlessly.

Some of the benefits of using Zod include:

-   [Simplicity]{#bf1a}
-   [Flexibility]{#42f0}
-   [Readability]{#31d0}
-   [Compatibility]{#e0a4}

Alright, to make it simple, I picked up a very easy to read API response and will use Playwright to create the test.

<figure>

</figure>

First of all, if you already have your *Playwright* project set up, let's create a test class. (If you have never set up Playwright, you can read my previous articles about it :) ).

    import { test, expect } from '@playwright/test';test('GET Yes / No', async ({ request }) => {
      const getAnswer = await request.get("https://yesno.wtf/api");
      expect(getAnswer.ok()).toBeTruthy();
    });

As you can see in the code block, that's a very simple API test that gets data from 'https://yesno.wtf/api' and checks the response is ok().

## Installing Zod {#6773}

Now.. we can start adding Zod to our project, as any other npm module:

    npm install zod

Once installed, we are going to create our Zod Schema. But before doing that, let's explore how the response from the API looks like under Network in DevTools.

<figure>

</figure>

As mentioned before, it is a very simple response with only 3 properties.

## Creating the Zod Schema {#5aa3}

So now, based on that, we can create our Zod Schema and include it in our test class:

    import { z } from "zod";const yesNorSchema = z.object({
      answer: z.string(),
      forced: z.boolean(),
      image: z.string()
    });export default yesNorSchema;

    import { test, expect } from '@playwright/test';
    import yesNorSchema from '../schemas/yesNoSchema'test('GET Yes / No', async ({ request }) => {
      const getAnswer = await request.get("https://yesno.wtf/api");
      expect(getAnswer.ok()).toBeTruthy();
    });

So.. what happens now ?

## Enhancing the Test with Zod Schema {#9a95}

Alright, we now need to validate that the response coming from the API has the same structure that our Zod Schema:

    test('GET Yes / No', async ({ request }) => {
      const getAnswer = await request.get("https://yesno.wtf/api");
      expect(getAnswer.ok()).toBeTruthy();
      // Get the answer in json() format
      const responseJson = await getAnswer.json();
      // Assert
      expect(() => yesNorSchema.parse(responseJson)).not.toThrow();
    });

**Done!**

As you can see, the assertion is checking that the function does not throw any error/exception.

**But .. what about the content?**

Oh! Sure, I did not forget about it, but let's do it step by step.

With Zod, we can also check that the value of a property in our Schema is correct without the need of updating our test / validation. So let's add that to our Schema:

    import { z } from "zod";const yesNorSchema = z.object({
      answer: z.string(),
      forced: z.boolean(),
      image: z.string()
    }).refine(obj => obj.answer === "yes", {message: "Response needs to be 'yes'"});export default yesNorSchema;

Simple example. The **.refine()** function allows us to provide custom validations. In this case, I am just telling the Schema to make sure the value of the "*answer"* property is equals to "*yes*", and also, a custom message to make it easier to identify the error.

## Running the Test {#fdf6}

Check what happens when executing the test!

<figure>

</figure>

**Few things to check here:**

1- I am printing the "*answer*" attribute, and as you can see in the console (green square), the answer was "*no*".

2- The custom error message stating the expected value should be '*yes*'.

On the other hand, when the "*answer*" value is yes, we can see the test passed!

<figure>

</figure>

**We are all set!**

## Conclusion {#5278}

Zod is a very nice and helpful tool to improve our testing. With intuitive syntax, it makes our validations easier and more readable with less effort to enforce some rules, leading to more secure/safe data transfer.

Very cool, isn't it?

As always, I hope you find this helpful and learn something from it!

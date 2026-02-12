---
layout: post
title: "TypeScript and React: A Guide to Stronger Code with Exhaustiveness Checking and Enums"
slug: building-robust-react-apps-typescript-enums-and-exhaustiveness-checking-in-action
date_published: 2023-12-17T15:07:05.000Z
date_updated: 2023-12-17T20:28:48.000Z
tags: Web Development
---

![](../assets/images/building-robust-react-apps/cover.jpeg)

This might be my first blog post in about 10 years. Coming back to blogging after such a long time is pretty exciting but also a bit stressful. The main reason I wanted to start blogging again is because I noticed I was always consuming stuff in my life but not really creating anything. I figured blogging is the easiest and quickest way for me to create something. So, what am I gonna write about in my first blog post?

Using TypeScript with React is like having a helpful friend for your coding. It checks for mistakes and gives smart ideas while you work. This makes your code stronger and makes big projects easier. We'll learn about [exhaustiveness checking](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking), how types narrow down, and when to use TypeScript enums along the way.

### Exhaustiveness Checking for Robust Code

One of the key features TypeScript brings to the table is exhaustiveness checking. Imagine building a React app with a `PaymentMethodSelector` component, and TypeScript ensuring that you handle all possible states. Let's explore this concept with a practical example.

### Not Handling Impossible State (Non-Exhaustive)

    // PaymentMethodSelector component file
    import React from 'react';
    
    export type PaymentMethod = 'CreditCard' | 'PayPal' | 'Bitcoin';
    
    interface PaymentMethodSelectorProps {
      selectedMethod: PaymentMethod;
    }
    
    const getPaymentMethod = (selectedMethod: PaymentMethod) => {
      switch (selectedMethod) {
        case "CreditCard":
          return <div>Select Credit Card as your payment method</div>;
        case "PayPal":
          return <div>Select PayPal as your payment method</div>;
        // Oops! We missed handling 'Bitcoin' case, TypeScript won't complain about the missing case
      }
    };
    
    const PaymentMethodSelector: React.FC<PaymentMethodSelectorProps> = ({ selectedMethod }) => {
      const paymentMethod = getPaymentMethod(selectedMethod);
    
      return paymentMethod;
    };
    
    // In your React component
    const CheckoutPage: React.FC = () => {
      const userPaymentMethod: PaymentMethod = "Bitcoin";
    
      return <PaymentMethodSelector selectedMethod={userPaymentMethod} />;
    };

Without exhaustiveness checking, it's easier to overlook states and introduce potential issues. TypeScript's strict checks act as a safety net, ensuring your code is comprehensive and reducing the likelihood of runtime errors, especially crucial in a real-life application where payment processing is involved.

### Handling Impossible State (Exhaustive)

    // CommonUtils.ts
    export const notReachable = (_: never): never => {
      console.error('should never be reached', _);
      throw new Error('should never be reached');
    };
    
    // PaymentMethodSelector component file
    import React from 'react';
    import { notReachable } from './CommonUtils';
    
    export type PaymentMethod = 'CreditCard' | 'PayPal' | 'Bitcoin';
    
    interface PaymentMethodSelectorProps {
      selectedMethod: PaymentMethod;
    }
    
    const getPaymentMethod = (selectedMethod: PaymentMethod) => {
      switch (selectedMethod) {
        case 'CreditCard':
          return <div>Select Credit Card as your payment method</div>;
        case 'PayPal':
          return <div>Select PayPal as your payment method</div>;
        case 'Bitcoin':
          return <div>Select Bitcoin as your payment method</div>;
        // TypeScript will raise an error if we miss a case OR if we add a new method to type, but not the switch
        default:
          return notReachable(selectedMethod);
      }
    };
    
    const PaymentMethodSelector: React.FC<PaymentMethodSelectorProps> = ({ selectedMethod }) => {
      const paymentMethod = getPaymentMethod(selectedMethod);
    
      return paymentMethod;
    };
    
    // In your React component
    const CheckoutPage: React.FC = () => {
      const userPaymentMethod: PaymentMethod = 'Bitcoin';
    
      return <PaymentMethodSelector selectedMethod={userPaymentMethod} />;
    };
    

In this example, TypeScript ensures that every possible payment method state is handled in the `PaymentMethodSelector` component, making the code more robust and preventing potential issues during the checkout process.

By centralizing this error-handling logic in the `notReachable` function, we created a consistent approach to deal with default cases. This makes our code more maintainable and provides clarity on how unexpected scenarios are handled across different components.

## Enhancing Readability: TypeScript Enums

In our example, without TypeScript Enums, the payment methods are represented as string literals. While this works, it lacks the benefits of a centralized and named constant approach. Now, let's refactor the example using TypeScript Enums to bring clarity and improve code readability.

    // CommonUtils.ts
    export const notReachable = (_: never): never => {
      console.error('should never be reached', _);
      throw new Error('should never be reached');
    };
    
    // PaymentMethodEnums.ts
    enum PaymentMethod {
      CreditCard = 'CreditCard',
      PayPal = 'PayPal',
      Bitcoin = 'Bitcoin',
    }
    
    export default PaymentMethodEnum;
    
    // PaymentMethodSelector component file
    import React from 'react';
    import PaymentMethodEnum from './PaymentMethodEnums';
    import { notReachable } from './CommonUtils.ts';
    
    interface PaymentMethodSelectorProps {
      selectedMethod: PaymentMethodEnum;
    }
    
    const getPaymentMethod = (selectedMethod: PaymentMethodEnum) => {
      switch (selectedMethod) {
        case PaymentMethodEnum.CreditCard:
          return <div>Select Credit Card as your payment method</div>;
        case PaymentMethodEnum.PayPal:
          return <div>Select PayPal as your payment method</div>;
        case PaymentMethodEnum.Bitcoin:
          return <div>Select Bitcoin as your payment method</div>;
        default:
          return notReachable(selectedMethod);
      }
    };
    
    const PaymentMethodSelector: React.FC<PaymentMethodSelectorProps> = ({ selectedMethod }) => {
      const paymentMethod = getPaymentMethod(selectedMethod);
    
      return paymentMethod;
    };
    
    // In your React component
    const CheckoutPage: React.FC = () => {
      const userPaymentMethod: PaymentMethodEnum = PaymentMethodEnum.Bitcoin;
    
      return <PaymentMethodSelector selectedMethod={userPaymentMethod} />;
    };
    

## The Another Ways

### 1. ts-pattern

Let's enhance our code further by integrating the [`ts-pattern`](https://github.com/gvergnaud/ts-pattern) library instead of using switch-case with our custom exhaustiveness checking function:

    // ... (Previous imports)
    
    // Importing ts-pattern library
    import { match } from 'ts-pattern';
    
    // PaymentMethodSelector component file with ts-pattern
    const PaymentMethodSelector: React.FC<PaymentMethodSelectorProps> = ({ selectedMethod }) => {
      const component = match(selectedMethod)
        .with(PaymentMethodEnum.CreditCard, () => <div>Select Credit Card as your payment method</div>)
        .with(PaymentMethodEnum.PayPal, () => <div>Select PayPal as your payment method</div>)
        .with(PaymentMethodEnum.Bitcoin, () => <div>Select Bitcoin as your payment method</div>)
        .exhaustive();
    
      return component;
    };
    

With `ts-pattern`, if your list of patterns doesn’t cover all edge cases of your input data-structure, it is able to tell you what cases are missing to make your branching exhaustive. Also, it infers a precise type for your input based on the patterns you provide. This ensures that your code is exhaustive and that TypeScript understands the refined types resulting from your pattern matching. Make sure to install the library (`npm install ts-pattern`) before using it in your project. Adjust the patterns and components according to your specific needs and use cases.

### 2. satisfies never

    // PaymentMethodEnums.ts
    enum PaymentMethod {
      CreditCard = 'CreditCard',
      PayPal = 'PayPal',
      Bitcoin = 'Bitcoin',
    }
    
    export default PaymentMethodEnum;
    
    // PaymentMethodSelector component file
    import React from 'react';
    import PaymentMethodEnum from './PaymentMethodEnums';
    
    interface PaymentMethodSelectorProps {
      selectedMethod: PaymentMethodEnum;
    }
    
    const getPaymentMethod = (selectedMethod: PaymentMethodEnum) => {
      switch (selectedMethod) {
        case PaymentMethodEnum.CreditCard:
          return <div>Select Credit Card as your payment method</div>;
      }
      // At this point, the selected method can only be "PayPal" or "Bitcoin"
      // Credit card has been eliminated in the previous step
      // Instead of creating a custom function, you can use 'satisfies never' for exhaustiveness checking
      selectedMethod satisfies never
    };
    
    const PaymentMethodSelector: React.FC<PaymentMethodSelectorProps> = ({ selectedMethod }) => {
      const paymentMethod = getPaymentMethod(selectedMethod);
    
      return paymentMethod;
    };
    
    // In your React component
    const CheckoutPage: React.FC = () => {
      const userPaymentMethod: PaymentMethodEnum = PaymentMethodEnum.PayPal;
    
      return <PaymentMethodSelector selectedMethod={userPaymentMethod} />;
    };
    

The another way to handle impossible states in a `switch` statement is TypeScript's 'satisfies never,' a powerful tool for exhaustiveness checking. Instead of creating a custom function like `notReachable`, you can directly use 'satisfies never' to ensure that all cases are covered. This approach simplifies your code while maintaining exhaustiveness and improving readability.

And that's it for today! 🎉 I hope you liked reading this and now feel a bit more sure about TypeScript's narrowing, exhaustiveness checking, and enums.

**Resources:**

> 'satisfies never' is SO GOOD for exhaustiveness checking in switch statements.
> 
> With this little annotation, you'll know you've checked every single branch of your union. Beautiful. [pic.twitter.com/24FnS95sCt](https://t.co/24FnS95sCt)
> — Matt Pocock (@mattpocockuk) [August 14, 2023](https://twitter.com/mattpocockuk/status/1691014244609765376?ref_src=twsrc%5Etfw)

- [Advanced typescript for React developers - part 3](https://www.developerway.com/posts/advanced-typescript-for-react-developers-part-3?ref=nurcin.dev)

- [Make Impossible States Impossible](https://kentcdodds.com/blog/make-impossible-states-impossible?ref=nurcin.dev)

- [Bringing Pattern Matching to TypeScript 🎨 Introducing TS-Pattern](https://dev.to/gvergnaud/bringing-pattern-matching-to-typescript-introducing-ts-pattern-v3-0-o1k?ref=nurcin.dev)
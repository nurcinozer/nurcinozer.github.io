---
layout: post
title: "TypeScript and React: Discriminated Unions When Fetching Data"
slug: typescript-discriminated-unions-when-fetching-data
date_published: 2024-06-01T08:40:23.000Z
date_updated: 2024-06-01T09:58:02.000Z
tags: Web Development
---

![](../assets/images/typescript-discriminated-unions/cover.jpeg)

As frontend developers, our job often involves handling states correctly. If you handle states with TypeScript discriminated union types, I can say that your job will become easier, and you will better understand the connection between data and UI. How? Let's first briefly discuss what discriminated union types are.

**Discriminated Union Types**

Discriminated unions are a powerful feature in TypeScript that allows you to create types representing multiple different types, each with its specific properties. You might have already noticed this when I explained exhaustiveness checking in [my previous post](https://www.nurcin.dev/posts/building-robust-react-apps-typescript-enums-and-exhaustiveness-checking-in-action).

Here's an example to illustrate how you can use a discriminated union to display a component in two slightly different ways.

Let's say you have two types of user profiles: **Admin** and **User**. Each profile has some common properties like **id** and **name**, but also has specific properties, like **permissions** for Admin and **subscriptionLevel** for User.

First, define the discriminated union type:

    // Define the discriminated union type for user profiles
    type UserProfile = 
      | { type: 'admin'; id: number; name: string; permissions: string[] }
      | { type: 'user'; id: number; name: string; subscriptionLevel: string };
    
    // Example user profiles
    const adminProfile: UserProfile = {
      type: 'admin',
      id: 1,
      name: 'Alice',
      permissions: ['manage-users', 'edit-content']
    };
    
    const userProfile: UserProfile = {
      type: 'user',
      id: 2,
      name: 'Bob',
      subscriptionLevel: 'premium'
    };

Next, create a React component that takes this UserProfile type and displays it appropriately based on its type:

    import React from 'react';
    
    interface UserProfileProps {
      profile: UserProfile;
    }
    
    const UserProfileComponent: React.FC<UserProfileProps> = ({ profile }) => {
      switch (profile.type) {
        case 'admin':
          return (
            <div>
              <h1>Admin Profile</h1>
              <p>ID: {profile.id}</p>
              <p>Name: {profile.name}</p>
              <p>Permissions: {profile.permissions.join(', ')}</p>
            </div>
          );
        case 'user':
          return (
            <div>
              <h1>User Profile</h1>
              <p>ID: {profile.id}</p>
              <p>Name: {profile.name}</p>
              <p>Subscription Level: {profile.subscriptionLevel}</p>
            </div>
          );
        default:
          return notReachable(profile.type); // You can read my previous post to understand what notReachable is
      }
    };

**What about when fetching data?**

One of the most common patterns frontend developers encounter is the "data/loading/error" pattern. When we handle this pattern with discriminated unions, you will see that we can avoid some problems. Without discriminated unions, everything is optional and available to everything else even if it doesn’t make sense: For example, you can access the error or data when loading is set to true.

Let's first examine the states we need:

- When type is loading, data or error are never present.
- When type is success, data is always present.
- When type is error, error is always present.

Here is how we can define and use these states:

    type FetchState<T> = 
      | { type: 'loading' }
      | { type: 'success'; data: T }
      | { type: 'error'; error: Error };

**Example: Fetching Products**

Let's consider an example where we need to fetch and display a list of products. Each product has the following properties: **id**, **name**, **price**, and **description**.

**Defining the Types**

First, define the types for the product data:

    type Product = {
      id: number;
      name: string;
      price: number;
      description: string;
    };
    

**Creating a GraphQL Query**

We'll use Apollo Client to fetch the products. Define the GraphQL query to fetch the products:

    import { gql } from '@apollo/client';
    
    const GET_PRODUCTS = gql`
      query GetProducts {
        products {
          id
          name
          price
          description
        }
      }
    `;

**Building the React Component**

Create a React component that uses Apollo Client to fetch the products and handle the different states:

    import React from 'react';
    import { useQuery } from '@apollo/client';
    
    const ProductList: React.FC = () => {
      const { loading, error, data } = useQuery(GET_PRODUCTS);
    
      const state: FetchState<Product[]> = loading
        ? { type: 'loading' }
        : error
        ? { type: 'error', error: error }
        : { type: 'success', data: data.products };
    
      switch (state.type) {
        case 'loading':
          return <div>Loading...</div>;
        case 'success':
          return (
            <ul>
              {state.data.map(product => (
                <li key={product.id}>
                  <h2>{product.name}</h2>
                  <p>{product.description}</p>
                  <p>${product.price.toFixed(2)}</p>
                </li>
              ))}
            </ul>
          );
        case 'error':
          return <div>Error: {state.error}</div>;
        default:
          return notReachable(state.type);
      }
    };
    
    export default ProductList;

We have defined a Product type representing the product data structure and a FetchState discriminated union type to manage the fetching states. Based on the query's result, we set the appropriate state (loading, success, or error) using discriminated unions.

**Creating a Common Hook**

We can create a common hook, useFetch, to handle data fetching with Apollo Client and TypeScript discriminated union types. This hook can be reused whenever we need to fetch data:

    import { useQuery, DocumentNode } from '@apollo/client';
    
    type FetchState<T> = 
      | { type: 'loading' }
      | { type: 'success'; data: T }
      | { type: 'error'; error: Error };
    
    const useFetch = <T>(query: DocumentNode): FetchState<T> => {
      const { loading, error, data } = useQuery<T>(query);
    
      if (loading) {
        return { type: 'loading' };
      }
    
      if (error) {
        return { type: 'error', error: error };
      }
    
      return { type: 'success', data: data as T };
    };

Now that we have the useFetch hook, let's see how to use it in a React component for any GraphQL query.

    const ProductList: React.FC = () => {
      const state = useFetch<Product[]>(GET_PRODUCTS);
    
      switch (state.type) {
        case 'loading':
          return <div>Loading...</div>;
        case 'success':
          return (
            <ul>
              {state.data.map(product => (
                <li key={product.id}>
                  <h2>{product.name}</h2>
                  <p>{product.description}</p>
                  <p>${product.price.toFixed(2)}</p>
                </li>
              ))}
            </ul>
          );
        case 'error':
          return <div>Error: {state.error}</div>;
        default:
          return notReachable(state.type);
      }
    };
    
    export default ProductList;
    

We have defined a useFetch hook that takes a GraphQL query as a parameter. This hook uses the useQuery hook from Apollo Client to execute the query and returns the appropriate state based on the query result.

By using TypeScript discriminated union types with Apollo Client, we create a more organized and error-resistant way to fetch data in React applications. This approach helps manage each state correctly, reducing mistakes and making the code easier to maintain.

### Resources
- [Advanced typescript for React developers - discriminated unions](https://www.developerway.com/posts/advanced-typescript-for-react-developers-discriminated-unions)

- [TypeScript Discriminated Unions for Frontend Developers](https://www.totaltypescript.com/discriminated-unions-are-a-devs-best-friend)

- [improved QueryResult type for better loading/error/data inference by Kumagor0 · Pull Request #6006 · apollographql/apollo-client](https://github.com/apollographql/apollo-client/pull/6006)

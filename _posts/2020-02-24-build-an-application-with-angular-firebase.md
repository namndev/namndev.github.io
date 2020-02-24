---
layout: post
title: "Building an application with Angular, Jest, Firebase"
image: https://www.nuget.org/profiles/firebase/avatar?imageSize=512
tags: [Angular, Jest, Firebase]
comments: true
---

I’ve written several posts in the past using Firebase and the [AngularFire2](https://github.com/angular/angularfire) library. The AngularFire2 library makes using and integrating Firebase with your Angular applications super fun and easy.

`AngularFire2` also enables you to build `JAMStack` applications which only require a frontend and calls to the various Firebase services (Auth, Database, etc.). After following the docs on [the AngularFire2 README](https://github.com/angular/angularfire/blob/master/docs/install-and-setup.md), you can get up and running fairly easily. From there its just a matter of injecting the different services into your Angular Components.

I recently built an Angular application that uses `AngularFire2`. The application also uses Jest for unit testing. I learned some in the process of building it and wanted to share for future reference.

This post is going to cover my app and some basics about setting up Jest. I’m not going to cover the initial setup of AngularFire2 since their GitHub repo covers it. I’m also not going to go over a lot about integrating Jest with Angular, except to say that I’m using an [Angular Builder](https://angular.io/guide/cli-builder) for Jest in lieu of Karma. Builders are great since they let you leverage the Angular CLI. I’ll cover briefly more on those and using Jest in the first section.

## Reyrey’s Restaurants

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2020/01/screen-shot-2020-01-07-at-4.31.47-am.png" />
</p>

The app that I’m going to cover is called “Reyrey’s Restaurants.” You can reach it by going to [www.reyreysrestaurants.com](https://www.reyreysrestaurants.com). The application is a fun way to keep track of restaurants you visit in your city. The project is built and hosted with Firebase, and built with [AngularFire2](https://github.com/angular/angularfire) to connect to the authentication and database services. I made it open source and you can check out [the source code on GitHub here](https://github.com/andrewevans0102/reyreys-restaurants).

Also, the reason I built the application was to have a fun way to track restaurants in my city and incorporate my cat (Rey) into one of my projects.0. I already have [Chessie Choochoo](https://github.com/andrewevans0102/chessie-choochoo) for my other cat (Chestnut) so I didn’t want to leave out Rey (checkout [www.chessiechoochoo.com](https://www.chessiechoochoo.com)).

I setup some docs so you can easily see how to use the app here. The basic premise is that you create an account, then add restaurants to a “wanna go” section. When you visit your restaurant you can “promote” it to “been there” and add a review with comments and stars etc. here are some screenshots:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2020/01/create2-collage.jpg" />
</p>


## Project Setup

As I mentioned in the start, the two big things with this project were [AngularFire2](https://github.com/angular/angularfire) and [Jest](https://jestjs.io/en/).

I’m not going to go into how to setup AngularFire2 with your project since the README in their repo pretty much covers it. I will, however, point you to my post on [How the AngularFire Library makes Firebase feel like Magic](https://indepth.dev/how-the-angularfire-library-makes-firebase-feel-like-magic/#angularfire--1) as that has a good set of instructions to get you started.

For setting up Jest with your project, there are a few ways to do this. I found using the [Jest Builder here was the easiest option for me](https://github.com/just-jeb/angular-builders/tree/master/packages/jest). Other than the instructions on the README, I also did the following:

* created a [`jest.config.js`](https://github.com/andrewevans0102/reyreys-restaurants/blob/master/jest.config.js) file based on the recommended example from the builder Repo
* [added a `–runInBand` option for the Jest CLI](https://github.com/andrewevans0102/reyreys-restaurants/commit/73cd47f9ce9ad6b26efdcaa17a7f37833baccc48#diff-2d0cd5d10b9604941c38c6aac608178a) to resolve memory errors I was seeing with my CircleCI build

The cool part about using a builder was that I was able to leverage the existing Angular CLI to do this work. So anytime I called `ng test` it would invoke the Jest test runner rather than the Karma runner that is normally the default.

After playing with it some I have to say I really liked Jest because of the following:

* The error messages and warnings were easy to understand
* The test runner gives you more fine grained options

I’m not really going to go into a lot about Jest because several other folks have covered this very well. I recommend reviewing the post [Angular CLI: “ng test” with Jest in 3 minutes (v2)](https://medium.com/angular-in-depth/search?q=jest). Also (event hough the article doesn’t use builders) I recommend checking out the article [Integrate Jest into an Angular application and library](https://medium.com/angular-in-depth/integrate-jest-into-an-angular-application-and-library-163b01d977ce) for more on Jest with Angular. Finally, the [Jest Getting Started Docs](https://jestjs.io/docs/en/getting-started) are a great place to go for examples and much more in depth information.

## TestinG Angularfire2 with jest

Normally, unit testing of libraries with different services was pretty straightforward. You mock the dependencies that you need to inject and use the various hooks (beforeEach, afterEach etc.) to handle the data you’re testing.

With AngularFire2 I had a number of issues trying to mock the different libraries because of the different methods that I needed to handle for my components etc. This wasn’t documented as much as I had hoped, and required a fairly extensive amount of googling. Fortunately, I found the [GitHub issue here](https://github.com/angular/angularfire/issues/18) that discusses adding docs on testing to the project repo. Within this GitHub issue [this response had a great example](https://github.com/angular/angularfire/issues/18#issuecomment-351670746) that helped me to learn how to do this for my project.

> As a small disclaimer, before I go into my tests I want to note that there are still a lot of tests that could be added. As of this writing, time constraints have limited how many I can add. I’ve built out a successful set of tests for the service classes in my project. The setup I used for these tests can be used for the components that rely on them as well.

I wrote a set of service classes that pull out the AngularFire2 services into their own classes. This made it easier because then I had more flexibility with naming and how I wanted to use AngularFire2.

The basic process to test these services is to create stub, and mock the values for the AngularFire2 library methods. These mock values then actually mock the real values that would be returned from the Firebase service methods.

For the [Authentication Service](https://github.com/andrewevans0102/reyreys-restaurants/blob/master/src/app/services/authentication/authentication.service.ts) I have the following setup:

```js
const credentialsMock = {
  email: 'abc@123.com',
  password: 'password'
};
 
const userMock = {
  uid: 'ABC123',
  email: credentialsMock.email
};
 
const createUserMock = {
  user: {
    uid: 'ABC123',
    email: credentialsMock.email
  }
};
 
const fakeAuthState = new BehaviorSubject(null);
 
const fakeSignInHandler = (email, password): Promise<any> => {
  fakeAuthState.next(userMock);
  return Promise.resolve(userMock);
};
 
const fakeCreateUserHandler = (email, password): Promise<any> => {
  fakeAuthState.next(createUserMock);
  return Promise.resolve(createUserMock);
};
 
const fakeSignOutHandler = (): Promise<any> => {
  fakeAuthState.next(null);
  return Promise.resolve();
};
 
const angularFireAuthStub = {
  authState: fakeAuthState,
  auth: {
    createUserWithEmailAndPassword: (email: string, password: string) =>
      fakeCreateUserHandler(email, password),
    signInWithEmailAndPassword: (email: string, password: string) =>
      fakeSignInHandler(email, password),
    signOut: () => fakeSignOutHandler()
  }
};
```

Then in my actual test describe block I reference the angularFireAuthStub value here:

```js
describe('AuthenticationService', () => {
  let service: AuthenticationService;
  let afAuth: AngularFireAuth;
 
  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [{ provide: AngularFireAuth, useValue: angularFireAuthStub }]
    });
 
    service = TestBed.get(AuthenticationService);
    afAuth = TestBed.get(AngularFireAuth);
  });
 
  afterEach(() => {
    fakeAuthState.next(null);
  });
```

Then in the tests themselves I just call my service methods and check for the mock and stub responses:

```js
test('should call the create user with email and password successfully', async () => {
  const response = await service.createUserWithEmailAndPassword(
    credentialsMock.email,
    credentialsMock.password
  );
  expect(response).toBe(createUserMock.user.uid);
});
```

Once I had the Authentication Service up and running I next built out the tests for the Database Service. The setup was similar to the Authentication Service and had the following:

```js
let service: DatabaseService;
let savedValues = [];
 
const user = {
  uid: 'ABC123',
  firstName: 'first',
  lastName: 'last',
  email: 'abc@123.com'
};
 
const wgRestaurant: WgRestaurant = {
  id: '1234',
  uid: 'abc123',
  name: 'name',
  link: 'link',
  description: 'description',
  recorded: 1234
};
 
const btRestaurant: BtRestaurant = {
  id: '1234',
  uid: '5678',
  name: 'restaurant name',
  description: 'restaurant description',
  location: 'restaurant location',
  link: 'restaurant link',
  stars: 5,
  review: 'restaurant review',
  recorded: 1234
};
 
const fakeAddValueHandler = (value: any): Promise<any> => {
  return Promise.resolve(savedValues.push(value));
};
 
const deleteAddedValueHandler = (): Promise<any> => {
  return Promise.resolve((savedValues = []));
};
 
const firestoreStub = {
  collection: (name: string) => ({
    doc: (id: string) => ({
      valueChanges: () => new BehaviorSubject({ foo: 'bar' }),
      set: (d: any) => fakeAddValueHandler(d)
    })
  }),
  createId: () => {
    return new Promise((resolve, reject) => resolve('1234567890'));
  },
  doc: (idFirst: string) => ({
    collection: (name: string) => ({
      doc: (idSecond: string) => ({
        valueChanges: () => new BehaviorSubject({ foo: 'bar' }),
        set: (d: any) => fakeAddValueHandler(d),
        delete: () => deleteAddedValueHandler()
      })
    })
  })
};
```

If you notice I’m just generically using an array whenever values are saved to the Cloud Firestore database. This obviously be customized for more fine grained testing. I was just really concerned with basic calling of the different methods here so I left it this way.

I used a beforeEach and afterEach to setup the tests as you see here:

```js
beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [{ provide: AngularFirestore, useValue: firestoreStub }]
  });
 
  service = TestBed.get(DatabaseService);
});
 
// clear out any saved values
afterEach(() => (savedValues = []));
```

Then, finally when the tests are actually called you just call the service methods and check for the stub and mock values:

```js
test('should call add user successfully', async () => {
  await service.addUser(user);
  expect(savedValues.length).toEqual(1);
});
```

I’ve had a lot of experience with Karma, so this was my first time really playing with Jest. Overall I found it very intuitive and it was fairly easy to work with. In particular I liked the warning and messages that the CLI gave me. They really helped to workout what I needed for configuration and building tests normally etc.

## Closing Thoughts

So I hope you enjoyed this post and you learned something from it as well. I really enjoy using AngularFire2 with my projects because it makes it easy to integrate Firebase into Angular apps. It was also cool to use Jest for unit testing instead of Karma has I had always done before. My project here really just covers some basics and there is a lot more you can do with both AngularFire2 and Jest. Also I hope you check out [ReyRey’s Restaurants](https://www.reyreysrestaurants.com/), and possibly even use it as you’re checking out local restaurants!

Thank you for reading!
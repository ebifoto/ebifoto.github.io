# NgRx in Depth

## Part I - Preparation

When the article is written, the recommended Node version is Node 12.
All the contents are provided by [Angular University](https://angular-university.io/).  
The recommended editor is [Visual Studio Code](https://code.visualstudio.com/download).
The main programming language to be used is `TypeScript`.

### Node Download Link

If you're new to Angular, Node is required for running `npm` and Angular.  
To install Node, or update to latest Node version, use the following link to download the latest package.  
[Node Official Website](https://nodejs.org/en/)

To check Node version, run the following command in command line.

```console
node -v
```

### Install Angular CLI

If you're new to Angular, install Angular CLI. `-g` means to install the CLI globally.

```console
npm install -g @angular/cli
```

### Clone and run the repository

This repo is used for through out the content as the practice material.

<https://github.com/angular-university/ngrx-course>

Once the repository is cloned, go to the repository directory and run command `code .` to open up the repository in VS Code.  
In VS Code Terminal, run `npm install` which will install all the packages required by the project.  

In the repository, there is a backend server written in Express located in `Server` directory. Run `npm run server` in command line to start the development backend server. To run Angular application, execute `npm start` in command line.

Backend runs at <http://localhost:9000/>, and Angular app runs at <http://localhost:4200/>.

## Extended Reading - What is NgRx

NgRx stands for "Angular Reactive Extensions". It's a state management solution for Angular ecosystem.

Without the state mangement, the application will hit the backend to fetch the data needed for the view every single time. This is because the observables are tightly binded with the components. When the components are disposed and re-created, the observable instances are initialized again and again, too.

By having the state management, we don't need to fetch the data that is not changed multiple times, and when the data is changed, we can immediately reflect the change on the interface without waiting for the response from the backend.

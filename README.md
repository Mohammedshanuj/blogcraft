# BlogProjects

### React blogging frontend with Markdown authoring and user authentication

BlogProjects is a frontend project for browsing and writing blog posts. It brings together React, Material UI, Redux Toolkit, client-side routing, and REST API integration to demonstrate a blogging application's user flows.

> **Repository status:** This repository contains a partial frontend source snapshot. The dependency manifest, several imported files, static assets, and backend are not included. It is not currently runnable from a fresh clone.

[Overview](#overview) · [Features](#features) · [Technology](#technology) · [Project structure](#project-structure) · [Routes](#routes) · [Getting started](#getting-started) · [API integration](#api-integration) · [Next steps](#next-steps)

## Overview

The application is organized around three areas:

- **Account access:** registration, login, logout, and password recovery screens.
- **Blog content:** active-post listings, personal-post listings, Markdown composition, image selection, detail views, editing, and likes.
- **User navigation:** standard, premium, and administrator destinations after login.

The backend is a separate service referenced by the frontend at `http://localhost:3001/api/v1`. Its implementation, database, and deployment configuration are outside this repository.

## Features

The following capabilities are represented in the source code; end-to-end behavior has not been verified.

| Area | Source implementation |
| --- | --- |
| Registration and login | Forms submit account details to user API endpoints. Login stores the returned user and token in Redux. |
| Protected navigation | `RequireAuthe` checks whether a user exists in Redux before rendering nested routes. |
| Password recovery | Forgot-password and reset-password forms call the backend; the reset token comes from the URL. |
| Blog listings | The home feed requests active content; the personal feed requests the signed-in user's posts. |
| Markdown authoring | `CreateBlog` provides a Markdown editor and preview, title formatting, and heading selection. |
| Images | The editor supports file selection and a local image preview, with multipart submission configured. |
| Blog details and editing | Detail and edit screens load a post by ID; editing uses a PUT request. |
| Likes | The feed submits a like request and uses the returned count. |
| Role destinations | Login redirects according to the returned `userType`. The admin dashboard contains static sample data; the premium page reuses feed components. |

Role-based redirects and client-side route checks do not establish backend authorization. The `/admin` route is currently outside the protected route wrapper.

## Technology

| Purpose | Libraries used in the source |
| --- | --- |
| User interface | React, JavaScript, JSX, CSS |
| Components and styling | Material UI, MUI Icons, Emotion |
| Navigation | React Router (`Routes`, `Route`, `Navigate`, `Outlet`) |
| Shared state | Redux Toolkit, React Redux |
| HTTP requests | Axios |
| Markdown editing and rendering | `@uiw/react-md-editor` |
| Additional form/editor code | Formik, Validator, Day.js, MUI date pickers, Slate, Draft.js, `react-rte` |

Additional libraries appear in alternate or experimental components and are not all used by the main routed screens. Exact dependency versions cannot be established because `package.json` and a lockfile are absent.

## Project structure

Files currently sit at the repository root rather than inside a committed `src/` directory.

| Path | Responsibility |
| --- | --- |
| [`index.js`](./index.js) | Creates the React root and wraps the app with Redux Provider and BrowserRouter. |
| [`App.js`](./App.js) | Defines routes and the shared navigation layout. |
| [`components/`](./components/) | Blog editor, feed, account screens, navigation, and reusable UI. |
| [`components/reg/`](./components/reg/) | Alternate signup form, validation helper, and form hook. |
| [`components/register/`](./components/register/) | Registration implementations, validators, and editor experiments. |
| [`pages/`](./pages/) | Home, user, premium, administrator, blog creation, and blog detail screens. |
| [`redux/store.js`](./redux/store.js) | Configures the authentication and password-token reducers. |
| [`redux/slice/authSlice.js`](./redux/slice/authSlice.js) | Stores login state, user, and token; exposes login/logout actions. |
| [`redux/slice/passSlicer.js`](./redux/slice/passSlicer.js) | Defines a password-token slice. |
| [`utils/axiosData.js`](./utils/axiosData.js) | Shared request configuration and a POST helper. |

### How the frontend works

1. `index.js` initializes React, routing, and the Redux store.
2. `App.js` chooses the screen for the current URL.
3. Account forms call the user API. Successful login dispatches `setCredentials`.
4. `RequireAuthe` allows protected screens when a Redux user is present; otherwise it redirects to `/home`.
5. Feed and detail components fetch blog content through Axios.
6. The authoring component collects title, Markdown, formatting, image, and token data, then submits a create or update request.

Authentication state is held in memory. No session restoration or persisted Redux store is implemented in the committed code.

## Routes

| Route | Screen | Frontend access |
| --- | --- | --- |
| `/` | Home and active blog feed | Public |
| `/home` | Redirect to `/` | Public |
| `/login` | Login | Public |
| `/register`, `/reg` | Registration using `RegField` | Public |
| `/form` | Alternate signup form | Public |
| `/forgotPassword` | Password recovery | Public |
| `/setPassword/:token` | Password reset | Public |
| `/user` | User feed | Requires Redux user |
| `/premium` | Premium destination using shared feed UI | Requires Redux user |
| `/blog` | Blog editor | Requires Redux user |
| `/myBlog` | Personal blog feed | Requires Redux user |
| `/details/:id` | Blog details | Requires Redux user |
| `/edit/:id` | Blog editing | Requires Redux user |
| `/admin` | Static administrator dashboard | No route guard currently applied |

## Getting started

### Inspect the source

```bash
git clone https://github.com/Mohammedshanuj/BlogProjects.git
cd BlogProjects
```

### Restore the application before running it

There is currently no supported install, development, build, or test command in this repository. Running `npm install` or `npm start` requires restoring the original application configuration first.

The following items are needed:

1. **Dependency and build configuration:** restore `package.json`, the matching lockfile, and the original bundler configuration or scripts.
2. **HTML entry:** restore the application HTML containing the `root` element used by `index.js`.
3. **Missing imports:** restore `App.css`, `index.css`, `reportWebVitals.js`, the referenced `assets/` files, and `components/register/RegisterPage` imported by `App.js`.
4. **Backend service:** provide a compatible API at `http://localhost:3001/api/v1`, or update the frontend request URLs.
5. **Integration checks:** confirm API payloads, token handling, multipart image uploads, and server-side CORS configuration.

Once these files are restored, use the scripts defined in the restored `package.json`. The source contains Create React App-style entry code, but that alone does not confirm the original build setup.

## API integration

These endpoints are referenced by the frontend. They describe the client integration, not a verified backend API specification.

**Current base URL:** `http://localhost:3001/api/v1`

| Method | Path | Frontend purpose |
| --- | --- | --- |
| POST | `/user/reg` | Register a user |
| POST | `/user/login` | Log in and receive user/token data |
| POST | `/user/forgotPassword` | Request password recovery |
| POST | `/user/setPassword` | Submit a new password and reset token |
| GET | `/blog/isActive` | Fetch active posts |
| POST | `/blog/myBlogs` | Fetch personal posts using a token in the body |
| POST | `/blog/content` | Create blog content |
| GET | `/blog/content/:id` | Fetch a post for viewing or editing |
| PUT | `/blog/content/:id` | Update a post |
| POST | `/blog/like` | Submit a like action |

The client reads response fields including `token` and `user` for login, `contentData` for listings, `content` for individual posts, and `c` for the like count. The main editor submits `title`, `variant`, `bold`, `italic`, `description`, `image`, and `token`.

API URLs are currently hardcoded across components. Moving them into one configured Axios client is a recommended follow-up; no environment-variable configuration is implemented yet.

## Next steps

These are proposed improvements, not completed features.

- [ ] Restore the missing project files and verify a clean installation and build.
- [ ] Consolidate alternate registration and editor implementations.
- [ ] Centralize API URLs, authentication handling, and error responses.
- [ ] Add role-aware frontend guards and enforce permissions on the backend.
- [ ] Fix editor create/edit fallback handling and validate required fields.
- [ ] Verify image-upload serialization and release temporary object URLs.
- [ ] Add loading, empty, success, and error states to API-driven screens.
- [ ] Improve layouts for mobile screens.
- [ ] Replace administrator sample data with real API integration.
- [ ] Add meaningful tests for authentication, authoring, and API failures.
- [ ] Add screenshots and a demo link after the application runs successfully.

## Validation

Documentation was checked against the committed source files and route definitions. Runtime, build, and end-to-end tests could not be performed because the application configuration, required imports, and backend are missing.

## Author

**Mohammed Shanuj** — [GitHub](https://github.com/Mohammedshanuj)

## License

No license file is currently included. Add an explicit license if you intend to grant reuse permissions.

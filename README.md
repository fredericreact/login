# Side Effect and UseEffect Hook

What is an effect or side effet ?

![image](https://user-images.githubusercontent.com/104289891/201330537-2d3d25ec-386e-4ed9-a2c5-36b08fb00c0d.png)


## What to add & Not to add as Dependencies

In the previous lecture, we explored useEffect() dependencies.

You learned, that you should add "everything" you use in the effect function as a dependency - i.e. all state variables and functions you use in there.

That is correct, but there are a few exceptions you should be aware of:

* You DON'T need to add state updating functions (as we did in the last lecture with setFormIsValid): React guarantees that those functions never change, hence you don't need to add them as dependencies (you could though)
* You also DON'T need to add "built-in" APIs or functions like fetch(), localStorage etc (functions and features built-into the browser and hence available globally): These browser APIs / global functions are not related to the React component render cycle and they also never change
* You also DON'T need to add variables or functions you might've defined OUTSIDE of your components (e.g. if you create a new helper function in a separate file): Such functions or variables also are not created inside of a component function and hence changing them won't affect your components (components won't be re-evaluated if such variables or functions change and vice-versa)

So long story short: You must add all "things" you use in your effect function if those "things" could change because your component (or some parent component) re-rendered. That's why variables or state defined in component functions, props or functions defined in component functions have to be added as dependencies!

Here's a made-up dummy example to further clarify the above-mentioned scenarios:

```javascript
import { useEffect, useState } from 'react';
 
let myTimer;
 
const MyComponent = (props) => {
 const [timerIsActive, setTimerIsActive] = useState(false);
 
 const { timerDuration } = props; // using destructuring to pull out specific props values
 
 useEffect(() => {
   if (!timerIsActive) {
     setTimerIsActive(true);
     myTimer = setTimeout(() => {
       setTimerIsActive(false);
     }, timerDuration);
   }
 }, [timerIsActive, timerDuration]);
};
```

In this example:
* **timerIsActive** is added as a dependency because it's component state that may change when the component changes (e.g. because the state was updated)
* **timerDuration** is added as a dependency because it's a prop value of that component - so it may change if a parent component changes that value (causing this MyComponent component to re-render as well)
* **setTimerIsActive** is NOT added as a dependency because it's that exception: State updating functions could be added but don't have to be added since React guarantees that the functions themselves never change
* **myTimer** is NOT added as a dependency because it's not a component-internal variable (i.e. not some state or a prop value) - it's defined outside of the component and changing it (no matter where) wouldn't cause the component to be re-evaluated
* **setTimeout** is NOT added as a dependency because it's a built-in API (built-into the browser) - it's independent from React and your components, it doesn't change



# Debouncing
On veut pas checker a chaque fois que l’user tape quelque chose sur le clavier, on prefere attendre qu’il ait fini de taper

# useReducer
useReducer is a more powerful state management than useState()

Par exemple quand je veux updater mon state mais que je depend de la previous version de 2 states : problem

![image](https://user-images.githubusercontent.com/104289891/201331654-0a9689a4-9e1d-4369-8b73-248a98785a65.png)

Peut combiner des states

On met le reducer function en dehors du componnent


```javascript
const emailReducer = (state, action) => {
  if(action.type ==='USER_INPUT') {
    return {
      value:action.val,
      isValid:action.val.includes('@')
    }
  }
 
  if(action.type ==='INPUT_BLUR') {
    return {
      value:state.value,
      isValid:state.value.includes('@')
    }
  }
 
return {
  value:'',
  isValid:false
}
}

```

```javascript
const Login = (props) => {
 
  const [enteredPassword, setEnteredPassword] = useState('');
  const [passwordIsValid, setPasswordIsValid] = useState();
  const [formIsValid, setFormIsValid] = useState(false);
 
 
const [emailState, dispatchEmail] = useReducer(emailReducer, {value:'', isValid:null });
 
  const emailChangeHandler = (event) => {
    dispatchEmail({type: 'USER_INPUT', val:event.target.value})
    setFormIsValid(
            event.target.value.includes('@') && enteredPassword.trim().length > 6
            );
   
  };
 
  const passwordChangeHandler = (event) => {
setEnteredPassword(event.target.value)
    setFormIsValid(
      emailState.isValid && event.target.value.trim().length > 6
      );
   
  };
 
  const validateEmailHandler = () => {
dispatchEmail({type:'INPUT_BLUR'})
  };
 
  const validatePasswordHandler = () => {
    setPasswordIsValid(enteredPassword.trim().length > 6);
  };
 
  const submitHandler = (event) => {
    event.preventDefault();
    props.onLogin(emailState.value, enteredPassword);
  };
 
  return (
    <Card className={classes.login}>
      <form onSubmit={submitHandler}>
        <div
          className={`${classes.control} ${
            emailState.isValid === false ? classes.invalid : ''
          }`}
        >
          <label htmlFor="email">E-Mail</label>
          <input
            type="email"
            id="email"
            value={emailState.value}
            onChange={emailChangeHandler}
            onBlur={validateEmailHandler}
          />
        </div>
        <div
          className={`${classes.control} ${
            passwordIsValid === false ? classes.invalid : ''
          }`}
        >
          <label htmlFor="password">Password</label>
          <input
            type="password"
            id="password"
            value={enteredPassword}
            onChange={passwordChangeHandler}
            onBlur={validatePasswordHandler}
          />
        </div>
        <div className={classes.actions}>
          <Button type="submit" className={classes.btn} disabled={!formIsValid}>
            Login
          </Button>
        </div>
      </form>
    </Card>
  );
};
 
export default Login;
```


## Adding Nested Properties As Dependencies To useEffect

In the previous lecture, we used object destructuring to add object properties as dependencies to useEffect().

```javascript
const { someProperty } = someObject;
useEffect(() => {
 // code that only uses someProperty ...
}, [someProperty]);
```

This is a very common pattern and approach, which is why I typically use it and why I show it here (I will keep on using it throughout the course).

I just want to point out, that they key thing is NOT that we use destructuring but that we pass specific properties instead of the entire object as a dependency.

We could also write this code and it would work in the same way.


```javascript
useEffect(() => {
 // code that only uses someProperty ...
}, [someObject.someProperty]);
```

This works just fine as well!

But you should avoid this code:

```javascript
useEffect(() => {
 // code that only uses someProperty ...
}, [someObject]);
```

Why?

Because now the effect function would re-run whenever ANY property of someObject changes - not just the one property (someProperty in the above example) our effect might depend on.

![image](https://user-images.githubusercontent.com/104289891/201332186-725e1485-142a-4a28-9d46-4175680da3b7.png)


# React Context

Probleme : passer props en props galere

On a un component-wide, “behind the scenes” state storage

React Context

![image](https://user-images.githubusercontent.com/104289891/201332344-c258b168-4693-4c42-9655-a598e989c452.png)


Create the context


```javascript
import React from 'react';

const AuthContext = React.createContext({
    isLoggedIn:false
});

export default AuthContext
```

Provide it by wrapping it around your components - then remove props from App and MainHeader


```javascript
import React, { useState, useEffect } from 'react';
 
import Login from './components/Login/Login';
import Home from './components/Home/Home';
import MainHeader from './components/MainHeader/MainHeader';
import AuthContext from './store/auth-context'
 
function App() {
 
 
 
  const [isLoggedIn, setIsLoggedIn] = useState(false);
 
 
 
useEffect(()=> {
  const storedUserLoggedInInformation = localStorage.getItem('isLoggedIn');
  if (storedUserLoggedInInformation==='1'){
    setIsLoggedIn(true);
  }
}, [])
 
  const loginHandler = (email, password) => {
    // We should of course check email and password
    // But it's just a dummy/ demo anyways
    localStorage.setItem('isLoggedIn','1');
    setIsLoggedIn(true);
  };
 
  const logoutHandler = () => {
    localStorage.removeItem('isLoggedIn');
    setIsLoggedIn(false);
  };
 
  return (
    <AuthContext.Provider
    value={{
      isLoggedIn:isLoggedIn,
    }}>
      <MainHeader  onLogout={logoutHandler} />
      <main>
        {!isLoggedIn && <Login onLogin={loginHandler} />}
        {isLoggedIn && <Home onLogout={logoutHandler} />}
      </main>
      </AuthContext.Provider>
 
  );
}
 
export default App;
```

```javascript
import React from 'react';
 
import Navigation from './Navigation';
import classes from './MainHeader.module.css';
 
const MainHeader = (props) => {
  return (
    <header className={classes['main-header']}>
      <h1>A Typical Page</h1>
      <Navigation  onLogout={props.onLogout} />
    </header>
  );
};
 
export default MainHeader;
 

```

Then finally, use context here : 


```javascript
import React, {useContext} from 'react';
import AuthContext from '../../store/auth-context'
import classes from './Navigation.module.css';
 
const Navigation = (props) => {
const ctx = useContext(AuthContext)
 
 
  return (
    <nav className={classes.nav}>
      <ul>
        {ctx.isLoggedIn && (
          <li>
            <a href="/">Users</a>
          </li>
        )}
        {ctx.isLoggedIn && (
          <li>
            <a href="/">Admin</a>
          </li>
        )}
        {ctx.isLoggedIn && (
          <li>
            <button onClick={props.onLogout}>Logout</button>
          </li>
        )}
      </ul>
    </nav>
  );
};
 
export default Navigation;
```

On peut faire de meme pour logout :


```javascript
import React, { useState, useEffect } from 'react';
 
import Login from './components/Login/Login';
import Home from './components/Home/Home';
import MainHeader from './components/MainHeader/MainHeader';
import AuthContext from './store/auth-context'
 
function App() {
 
 
 
  const [isLoggedIn, setIsLoggedIn] = useState(false);
 
 
 
useEffect(()=> {
  const storedUserLoggedInInformation = localStorage.getItem('isLoggedIn');
  if (storedUserLoggedInInformation==='1'){
    setIsLoggedIn(true);
  }
}, [])
 
  const loginHandler = (email, password) => {
    // We should of course check email and password
    // But it's just a dummy/ demo anyways
    localStorage.setItem('isLoggedIn','1');
    setIsLoggedIn(true);
  };
 
  const logoutHandler = () => {
    localStorage.removeItem('isLoggedIn');
    setIsLoggedIn(false);
  };
 
  return (
    <AuthContext.Provider
    value={{
      isLoggedIn:isLoggedIn,
      onLogout:logoutHandler
    }}>
      <MainHeader   />
      <main>
        {!isLoggedIn && <Login onLogin={loginHandler} />}
        {isLoggedIn && <Home onLogout={logoutHandler} />}
      </main>
      </AuthContext.Provider>
 
  );
}
 
export default App;
```

```javascript
import React from 'react';
 
import Navigation from './Navigation';
import classes from './MainHeader.module.css';
 
const MainHeader = (props) => {
  return (
    <header className={classes['main-header']}>
      <h1>A Typical Page</h1>
      <Navigation   />
    </header>
  );
};
 
export default MainHeader;

```

```javascript
import React, {useContext} from 'react';
import AuthContext from '../../store/auth-context'
import classes from './Navigation.module.css';
 
const Navigation = () => {
const ctx = useContext(AuthContext)
 
 
  return (
    <nav className={classes.nav}>
      <ul>
        {ctx.isLoggedIn && (
          <li>
            <a href="/">Users</a>
          </li>
        )}
        {ctx.isLoggedIn && (
          <li>
            <a href="/">Admin</a>
          </li>
        )}
        {ctx.isLoggedIn && (
          <li>
            <button onClick={ctx.onLogout}>Logout</button>
          </li>
        )}
      </ul>
    </nav>
  );
};
 
export default Navigation;
```

On peut garder props pour les componeents qui tilisent directement les props ou ya pas de prosp chain.

On utilise surtout context quand ya des props chains quand les compoenents utilisent pas directement les props.


## Building & Using a Custom Context Provider Component


```javascript
import React, {useState, useEffect} from 'react';
 
const AuthContext = React.createContext({
    isLoggedIn:false,
    onLogout: () =>{},
    onLogin:(email,password)=>{}
});
 
export const AuthContextProvider = (props) => {
    const [isLoggedIn, setIsLoggedIn] = useState(false);
 
    useEffect(()=> {
        const storedUserLoggedInInformation = localStorage.getItem('isLoggedIn');
        if (storedUserLoggedInInformation==='1'){
          setIsLoggedIn(true);
        }
      }, [])
 
    const logoutHandler = () =>{
        localStorage.removeItem('isLoggedIn');
        setIsLoggedIn(false);
    }
 
    const loginHandler =()=>{
        localStorage.setItem('isLoggedIn','1');
        setIsLoggedIn(true)
    }
 
    return <AuthContext.Provider
    value={{
        isLoggedIn : isLoggedIn,
        onLogout: logoutHandler,
        onLogin: loginHandler
    }}
    >{props.children}</AuthContext.Provider>
}
 
export default AuthContext
```


```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
 
import './index.css';
import App from './App';
import { AuthContextProvider } from './store/auth-context';
 
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<AuthContextProvider><App /></AuthContextProvider>);
```

Use Context in App


```javascript
import React , {useContext} from'react';
 
import Login from './components/Login/Login';
import Home from './components/Home/Home';
import MainHeader from './components/MainHeader/MainHeader';
import AuthContext from './store/auth-context';
 
 
function App() {
 
 
const ctx = useContext(AuthContext);
 
  return (
 <React.Fragment>
      <MainHeader   />
      <main>
        {!ctx.isLoggedIn && <Login  />}
        {ctx.isLoggedIn && <Home  />}
      </main>
      </React.Fragment>
 
  );
}
 
export default App;
```

Use Context in Home and Login


```javascript
import React, {useContext} from 'react';
import AuthContext from '../../store/auth-context';
import Button from '../UI/Button/Button'
import Card from '../UI/Card/Card';
import classes from './Home.module.css';
 
const Home = (props) => {
  const authCtx = useContext(AuthContext)
  return (
    <Card className={classes.home}>
      <h1>Welcome back!</h1>
      <Button onClick={authCtx.onLogout}>Logout</Button>
    </Card>
  );
};
 
export default Home;
```

```javascript
import React, { useState, useEffect, useReducer, useContext } from 'react';
 
import Card from '../UI/Card/Card';
import classes from './Login.module.css';
import Button from '../UI/Button/Button';
import AuthContext from '../../store/auth-context';
 
const emailReducer = (state, action) => {
  if(action.type ==='USER_INPUT') {
    return {
      value:action.val,
      isValid:action.val.includes('@')
    }
  }
 
  if(action.type ==='INPUT_BLUR') {
    return {
      value:state.value,
      isValid:state.value.includes('@')
    }
  }
 
return {
  value:'',
  isValid:false
}
}
 
const passwordReducer =(state, action) =>{
 
 
  if(action.type ==='USER_INPUT') {
    return {
      value:action.val,
      isValid:action.val.trim().length > 6
    }
  }
 
  if(action.type ==='INPUT_BLUR') {
    return {
      value:state.value,
      isValid:state.value.trim().length > 6
    }
  }
 
return {
  value:'',
  isValid:false
}
 
}
 
const Login = (props) => {
  // const [enteredEmail, setEnteredEmail] = useState('');
  // const [emailIsValid, setEmailIsValid] = useState();
  // const [enteredPassword, setEnteredPassword] = useState('');
  // const [passwordIsValid, setPasswordIsValid] = useState();
  const [formIsValid, setFormIsValid] = useState(false);
 
 
const [emailState, dispatchEmail] = useReducer(emailReducer, {value:'', isValid:null });
 
const [passwordState, dispatchPassword] = useReducer (passwordReducer, {value:'', isValid:null })
 
const authCtx=useContext(AuthContext)
 
useEffect(()=>{
  console.log('effect running')
  return () => {
    console.log('effect cleanup')
  }
},[])
 
 
const {isValid:emailIsValid} =emailState;
const {isValid:passwordIsValid} = passwordState;
 
useEffect (()=>{
 
  const identifier = setTimeout(()=>{
 
    setFormIsValid(
      emailIsValid && passwordIsValid
      );
  }, 500)
 
return () => {
  clearTimeout(identifier)
}
 
}, [emailIsValid,passwordIsValid])
 
  const emailChangeHandler = (event) => {
    dispatchEmail({type: 'USER_INPUT', val:event.target.value})
    // setFormIsValid(
    //         event.target.value.includes('@') && passwordState.isValid
    //         );
   
  };
 
  const passwordChangeHandler = (event) => {
dispatchPassword({type:'USER_INPUT', val: event.target.value})
    // setFormIsValid(
    //   emailState.isValid && event.target.value.trim().length > 6
    //   );
   
  };
 
  const validateEmailHandler = () => {
dispatchEmail({type:'INPUT_BLUR'})
  };
 
  const validatePasswordHandler = () => {
dispatchPassword({type:'INPUT_BLUR'})
  };
 
  const submitHandler = (event) => {
    event.preventDefault();
    authCtx.onLogin(emailState.value, passwordState.value);
  };
 
  return (
    <Card className={classes.login}>
      <form onSubmit={submitHandler}>
        <div
          className={`${classes.control} ${
            emailState.isValid === false ? classes.invalid : ''
          }`}
        >
          <label htmlFor="email">E-Mail</label>
          <input
            type="email"
            id="email"
            value={emailState.value}
            onChange={emailChangeHandler}
            onBlur={validateEmailHandler}
          />
        </div>
        <div
          className={`${classes.control} ${
            passwordState.isValid === false ? classes.invalid : ''
          }`}
        >
          <label htmlFor="password">Password</label>
          <input
            type="password"
            id="password"
            value={passwordState.value}
            onChange={passwordChangeHandler}
            onBlur={validatePasswordHandler}
          />
        </div>
        <div className={classes.actions}>
          <Button type="submit" className={classes.btn} disabled={!formIsValid}>
            Login
          </Button>
        </div>
      </form>
    </Card>
  );
};
 
export default Login;
```

React context limitations

It’s good for app-wide / component wide state = state that affect multiple components

Not good for component configuration : for example my button which is reusable , used for login and logout

![image](https://user-images.githubusercontent.com/104289891/201332978-5f48e33a-93c3-420c-b186-356f5da0a852.png)


Refactoring input


```javascript
import React from 'react';
import classes from './Input.module.css'
const Input = (props) =>{
return (
    <div
          className={`${classes.control} ${
            props.isValid === false ? classes.invalid : ''
          }`}
        >
          <label htmlFor={props.id}>{props.label}</label>
          <input
            type={props.type}
            id={props.id}
            value={props.value}
            onChange={props.onChange}
            onBlur={props.onBlur}
          />
        </div>
)
}
 
export default Input;

```

```javascript
import React, { useState, useEffect, useReducer, useContext } from 'react';
 
import Card from '../UI/Card/Card';
import classes from './Login.module.css';
import Button from '../UI/Button/Button';
import AuthContext from '../../store/auth-context';
import Input from '../UI/Input/Input'
 
 
const emailReducer = (state, action) => {
  if(action.type ==='USER_INPUT') {
    return {
      value:action.val,
      isValid:action.val.includes('@')
    }
  }
 
  if(action.type ==='INPUT_BLUR') {
    return {
      value:state.value,
      isValid:state.value.includes('@')
    }
  }
 
return {
  value:'',
  isValid:false
}
}
 
const passwordReducer =(state, action) =>{
 
 
  if(action.type ==='USER_INPUT') {
    return {
      value:action.val,
      isValid:action.val.trim().length > 6
    }
  }
 
  if(action.type ==='INPUT_BLUR') {
    return {
      value:state.value,
      isValid:state.value.trim().length > 6
    }
  }
 
return {
  value:'',
  isValid:false
}
 
}
 
const Login = (props) => {
  // const [enteredEmail, setEnteredEmail] = useState('');
  // const [emailIsValid, setEmailIsValid] = useState();
  // const [enteredPassword, setEnteredPassword] = useState('');
  // const [passwordIsValid, setPasswordIsValid] = useState();
  const [formIsValid, setFormIsValid] = useState(false);
 
 
const [emailState, dispatchEmail] = useReducer(emailReducer, {value:'', isValid:null });
 
const [passwordState, dispatchPassword] = useReducer (passwordReducer, {value:'', isValid:null })
 
const authCtx=useContext(AuthContext)
 
useEffect(()=>{
  console.log('effect running')
  return () => {
    console.log('effect cleanup')
  }
},[])
 
 
const {isValid:emailIsValid} =emailState;
const {isValid:passwordIsValid} = passwordState;
 
useEffect (()=>{
 
  const identifier = setTimeout(()=>{
 
    setFormIsValid(
      emailIsValid && passwordIsValid
      );
  }, 500)
 
return () => {
  clearTimeout(identifier)
}
 
}, [emailIsValid,passwordIsValid])
 
  const emailChangeHandler = (event) => {
    dispatchEmail({type: 'USER_INPUT', val:event.target.value})
    // setFormIsValid(
    //         event.target.value.includes('@') && passwordState.isValid
    //         );
   
  };
 
  const passwordChangeHandler = (event) => {
dispatchPassword({type:'USER_INPUT', val: event.target.value})
    // setFormIsValid(
    //   emailState.isValid && event.target.value.trim().length > 6
    //   );
   
  };
 
  const validateEmailHandler = () => {
dispatchEmail({type:'INPUT_BLUR'})
  };
 
  const validatePasswordHandler = () => {
dispatchPassword({type:'INPUT_BLUR'})
  };
 
  const submitHandler = (event) => {
    event.preventDefault();
    authCtx.onLogin(emailState.value, passwordState.value);
  };
 
  return (
    <Card className={classes.login}>
      <form onSubmit={submitHandler}>
        <Input
        id="email"
        label="E-mail"
        type="email"
        isValid={emailIsValid}
        value={emailState.value}
        onChange={emailChangeHandler}
        onBlur={validateEmailHandler}
         />
 
        <Input
        id="password"
        label="Password"
        type="password"
        isValid={passwordIsValid}
        value={passwordState.value}
        onChange={passwordChangeHandler}
        onBlur={validatePasswordHandler}
         />
         
     
        <div className={classes.actions}>
          <Button type="submit" className={classes.btn} disabled={!formIsValid}>
            Login
          </Button>
        </div>
      </form>
    </Card>
  );
};
 
export default Login;
```

Ici dans cet exemple , utiliser props est mieux que context car mon Input component est reusable et est configure par les props, utiliser context api aurait pas ete bien
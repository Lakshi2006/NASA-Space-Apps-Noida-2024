# NASA-Space-Apps-Noida-2024
Project based on Leveraging Earth Observation Data for Informed Agricultural Decision Making
Technology Stack:

Frontend: React.js with Redux for state management
Backend: Node.js with Express.js for API routing
Database: MongoDB for storing user data and features

Folder Structure:
agriaura/
client/
public/
index.html
src/
components/
Dashboard.js
Feature.js
Login.js
SearchBar.js
containers/
App.js
reducers/
index.js
userReducer.js
featureReducer.js
actions/
userActions.js
featureActions.js
store.js
index.js
server/
models/
User.js
Feature.js
routes/
auth.js
features.js
app.js
package.json

Client-side Code:
'client/src/components/Login.js'
import React, { useState } from 'react';
import { connect } from 'react-redux';
import { login } from '../actions/userActions';

const Login = ({ login, history }) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (event) => {
    event.preventDefault();
    login(username, password);
  };

  return (
    <div>
      <h1>Agriaura</h1>
      <p>Namaste!</p>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={username}
          onChange={(event) => setUsername(event.target.value)}
          placeholder="USERNAME"
        />
        <input
          type="password"
          value={password}
          onChange={(event) => setPassword(event.target.value)}
          placeholder="PASSWORD"
        />
        <button type="submit">LOG IN</button>
        <p>or</p>
        <button>Sign in with Google</button>
        <a href="#" id="forgot-password">
          Forgot Password? Click here.
        </a>
      </form>
    </div>
  );
};

export default connect(null, { login })(Login);

'client/src/components/Dashboard.js'
import React from 'react';
import { connect } from 'react-redux';
import { getFeatures } from '../actions/featureActions';

const Dashboard = ({ features, getFeatures }) => {
  const [searchQuery, setSearchQuery] = useState('');

  useEffect(() => {
    getFeatures();
  }, []);

  const handleSearch = (event) => {
    setSearchQuery(event.target.value);
  };

  return (
    <div>
      <h1>Agridura</h1>
      <p>Welcome back, ABC!</p>
      <p>Free User Since 2024</p>
      <button>Upgrade to Premium</button>
      <h2>Features</h2>
      <ul>
        {features.map((feature) => (
          <li key={feature._id}>{feature.name}</li>
        ))}
      </ul>
      <button>Discover</button>
      <input
        type="search"
        value={searchQuery}
        onChange={handleSearch}
        placeholder="Search here"
      />
    </div>
  );
};

export default connect((state) => ({ features: state.features }), { getFeatures })(
  Dashboard
);

'client/src/reducers/index.js'
import { combineReducers } from 'redux';
import userReducer from './userReducer';
import featureReducer from './featureReducer';

const rootReducer = combineReducers({
  user: userReducer,
  features: featureReducer,
});

export default rootReducer;

'client/src/actions/userActions.js'
import axios from 'axios';

export const login = (username, password) => {
  return (dispatch) => {
    axios.post('/api/auth/login', { username, password })
      .then((response) => {
        dispatch({ type: 'LOGIN_SUCCESS', payload: response.data });
      })
      .catch((error) => {
        dispatch({ type: 'LOGIN_ERROR', payload: error.message });
      });
  };
};

'client/src/actions/featureActions.js'
import axios from 'axios';

export const getFeatures = () => {
  return (dispatch) => {
    axios.get('/api/features')
      .then((response) => {
        dispatch({ type: 'GET_FEATURES_SUCCESS', payload: response.data });
      })
      .catch((error) => {
        dispatch({ type: 'GET_FEATURES_ERROR', payload: error.message });
      });
  };
};


Server-side Code:
'server/models/User.js'
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: String,
  password: String,
});

const User = mongoose.model('User', userSchema);

module.exports = User;

'server/models/Feature.js'
const mongoose = require('mongoose');

const featureSchema = new mongoose.Schema({
  name: String,
});

const Feature = mongoose.model('Feature', featureSchema);

module.exports = Feature;

'server/routes/auth.js'
const express = require('express');
const router = express.Router();
const User = require('../models/User');

router.post('/login', (req, res) => {
  const { username, password } = req.body;
  User.findOne({ username }, (err, user) => {
    if (err) {
      res.status(500).send({ message: 'Error logging in' });
    } else if (!user) {
      res.status(401).send({ message: 'Invalid username or password' });
    } else {
      user.comparePassword(password, (err, isMatch) => {
        if (err) {
          res.status(500).send({ message: 'Error logging in' });
        } else if (!isMatch) {
          res.status(401).send({ message: 'Invalid username or password' });
        } else {
          res.send({ message: 'Logged in successfully' });
        }
      });
    }
  });
});

module.exports = router;

'server/routes/features.js'
const express = require('express');
const router = express.Router();
const Feature = require('../models/Feature');

router.get('/', (req, res) => {
  Feature.find({}, (err, features) => {
    if (err) {
      res.status(500).send({ message: 'Error fetching features' });
    } else {
      res.send(features);
    }
  });
});

module.exports = router;

'server/app.js'
const express = require('express');
const app = express();
const authRouter = require('./routes/auth');
const featureRouter = require('./routes/features');

app.use(express.json());
app.use('/api/auth', authRouter);
app.use('/api/features', featureRouter);

app.listen(3000, () => {
  console.log('Server listening on port 3000');
});


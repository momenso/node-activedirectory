ActiveDirectory for Node
=========

ActiveDirectory is an ldapjs client for authN (authentication) and authZ (authorization) for Microsoft Active Directory with range retrieval support for large Active Directory installations. This code was a port of an existing C# library (not published) that I had written a few years ago. Here are the key features

  - Authenticate
  - Authorization (via group membership information)
  - Nested groups support
  - Range specifier / retrieval support (http://msdn.microsoft.com/en-us/library/dd358433.aspx)

Required Libraries
-----------

ActiveDirectory uses the following additional node modules:

* [underscore] - a utility-belt library for JavaScript that provides a lot of the functional programming support
* [async] - Async utilities for node and the browser
* [ldapjs] - a pure JavaScript, from-scratch framework for implementing LDAP clients and servers in Node.js

Installation
--------------

```sh
npm install activedirectory
```

Usage
--------------

```js
var ActiveDirectory = require('activedirectory');
var ad = new ActiveDirectory(url, baseDN, username, password);
```

Optionally the configuration can be specified with an object:

```js
var ActiveDirectory = require('activedirectory');
var ad = new ActiveDirectory({ url: 'ldap://dc.domain.com',
                               baseDN: 'dc=domain,dc=com',
                               username: 'username@domain.com',
                               password: 'password' };);
```

The username and password specified in the configuration are what are used for user and group lookup operations.

---------------------------------------

Documentation
--------------
* [authenticate](#authenticate)
* [findUser](#findUser)
* [findGroup](#findGroup)
* [isUserMemberOf](#isUserMemberOf)
* [groupExists](#groupExists)
* [userExists](#userExists)
* [getGroupMembershipForGroup](#getGroupMembershipForGroup)
* [getGroupMembershipForUser](#getGroupMembershipForUser)
* [getUsersForGroup](#getUsersForGroup)

---------------------------------------

<a name="authenticate" />
### authenticate(username, password, callback)

Authenticates the username and password by doing a simple bind with the specified credentials.

__Arguments__

* username - The username to authenticate.
* password - The password to use for authentication.
* callback(err, authenticated) - A callback which is called after authentication is completed.

__Example__

```js
var ad = new ActiveDirectory(config);
var username = 'john.smith';
var password = 'password';

ad.authenticate(username, password, function(err, auth) {
  if (err) {
    console.log('ERROR: '+JSON.stringify(err));
    return;
  }
  
  if (auth) {
    console.log('Authenticated!');
  }
  else {
    console.log('Authentication failed!');
  }
});
```

---------------------------------------

<a name="findUser" />
### findUser(opts, username, includeMembership, callback)

Looks up or finds a username by their sAMAccountName, userPrincipalName, distinguishedName (dn) or custom filter. If found, the returned object contains all of the requested attributes. By default, the following attributes are returned: 

* userPrincipalName, sAMAccountName, mail, lockoutTime, whenCreated, pwdLastSet, userAccountControl, employeeID,  sn,  givenName, initials, cn, displayName, comment, description

__Arguments__

* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* username - The username to retrieve information about. Optionally can pass in the distinguishedName (dn) of the user to retrieve.
* includeMembership - Indicates if the request should also retrieve the group memberships for the user. Default = false;
* callback(err, user) - The callback to execute when completed. callback(err: {Object}, user: {User})

__Example__

```js
// Any of the following username types can be searched on
var sAMAccountName = 'username';
var userPrincipalName = 'username@domain.com';
var dn = 'CN=Smith\\, John,OU=UUsers,DC=domain,DC=com';

// Find user by a sAMAccountName
var ad = new ActiveDirectory(config);
ad.findUser(sAMAccountName, function(err, user) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! user) console.log('User: ' + sAMAccountName + ' not found.');
  else console.log(JSON.stringify(user));
});
```

---------------------------------------

<a name="findGroup" />
### findGroup(opts, groupName, callback)

Looks up or find a group by common name (CN) which is required to be unique in Active Directory or optionally by the distinguished name. Supports groups with range retrieval specifiers. The following attributes are returned by default for the group:

* objectCategory, distinguishedName, cn, description, member

__Arguments__

* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* groupName -  The group (cn) to retrieve information about. Optionally can pass in the distinguishedName (dn) of the group to retrieve.
* callback(err, group) - The callback to execute when completed. callback(err: {Object}, group: {Group})


__Example__

```js
// Any of the following group names can be searched on
var groupName = 'Employees';
var dn: 'CN=Employees,OU=Groups,DC=domain,DC=com'

// Find group by common name
var ad = new ActiveDirectory(config);
ad.findGroup(groupName, function(err, group) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! user) console.log('Group: ' + groupName + ' not found.');
  else {
    console.log(group);
    console.log('Members: ' + (group.member || []).length);
  }
});
```

---------------------------------------

<a name="isUserMemberOf" />
### isUserMemberOf(opts, username, groupName, callback)

Checks to see if a user is a member of the specified group. This function will also check for group membership inside of a group.  Even if a user is not explicity listed as a member of a particular group, if a group that the user is a member of belongs to the group, then this function will return true.

__Arguments__
* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* username - The username to check for membership. Can be specied as a sAMAccountName, userPrincipalName or distinguishedName (dn)
* groupName - The group to check for membership. Can be a commonName (cn) or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, result: {Boolean})

__Example__

```js
var username = 'user@domain.com';
var groupName = 'Employees';

var ad = new ActiveDirectory(config);
var ad.isUserMemberOf(username, groupName, function(err, isMember) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log(username + ' isMemberOf ' + groupName + ': ' + isMember);
});
```

---------------------------------------

<a name="groupExists" />
### groupExists(opts, groupName, callback)

Checks to see if the specified group exists.

__Arguments__
* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* groupName - The group to check if is defined. Can be a commonName (cn) or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, result: {Boolean})

__Example__

```js
var groupName = 'Employees';

var ad = new ActiveDirectory(config);
ad.groupExists(groupName, function(err, exists) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log(groupName + ' exists: ' + exists);
});
```

---------------------------------------

<a name="userExists" />
### userExists(opts, username, callback)

Checks to see if the specified user exists.

__Arguments__
* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* username - The username to check if it exists. Can be a sAMAccountName, userPrincipalName or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, result: {Boolean})

__Example__

```js
var username = 'john.smith';

var ad = new ActiveDirectory(config);
ad.userExists(username, function(err, exists) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log(username + ' exists: ' + exists);
});
```

---------------------------------------

<a name="getUsersForGroup" />
### getUsersForGroup(opts, groupName, callback)

For the specified group, retrieve all of the users that belong to the group. If the group contains groups, then the members of those groups are recursively retrieved as well to build a complete list of users that belong to the specified group.

__Arguments__
* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* groupName - The name of the group to retrieve membership from. Can be a commonName (cn) or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[User]})

__Example__

```js
var groupName = 'Employees';

var ad = new ActiveDirectory(config);
ad.getUsersForGroup(groupName, function(err, users) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! users) console.log('Group: ' + groupName + ' not found.');
  else {
    console.log(JSON.stringify(users));
  }
});
```


---------------------------------------

<a name="getGroupMembershipForUser" />
### getGroupMembershipForUser(opts, username, callback)

For the specified username, retrieve all of the groups that a user belongs to. If a retrieved group is a member of another group, then that group is recursively retrieved as well to build a complete hierarchy of groups that a user belongs to.

__Arguments__
* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* username - The name of the user to retrieve group membership for. Can be a sAMAccountName, userPrincipalName, or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[Group]})

__Example__

```js
var sAMAccountName = 'john.smith';

var ad = new ActiveDirectory(config);
ad.getGroupMembershipForUser(sAMAccountName, function(err, groups) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! groups) console.log('User: ' + sAMAccountName + ' not found.');
  else console.log(JSON.stringify(groups));
});
```

---------------------------------------

<a name="getGroupMembershipForGroup" />
### getGroupMembershipForGroup(opts, groupName, callback)

For the specified group, retrieve all of the groups that the group is a member of. If a retrieved group is a member of another group, then that group is recursively retrieved as well to build a complete hierarchy of groups that a user belongs to.

__Arguments__
* opts - Optional LDAP query string parameters to execute. { scope: '', filter: '', attributes: [ '', '', ... ], sizeLimit: 0, timelimit: 0 }
* groupName - The name of the user to retrieve group membership for. Can be a sAMAccountName, userPrincipalName, or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[Group]})

__Example__

```js
var groupName = 'Employees';

var ad = new ActiveDirectory(config);
ad.getGroupMembershipForGroup(groupName, function(err, groups) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! groups) console.log('Group: ' + groupName + ' not found.');
  else console.log(JSON.stringify(groups));
});
```


  [underscore]: http://underscorejs.org/
  [async]: https://github.com/caolan/async
  [ldapjs]: http://ldapjs.org/
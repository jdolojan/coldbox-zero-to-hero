(All commands assume we are in the `box` shell unless stated otherwise.)

1.  Create a folder for your app on your hard drive called `soapbox`.
2.  Scaffold out a new Coldbox application with TestBox included.

```sh
coldbox create app soapbox --installTestBox
```

3.  Start up a local server

```sh
start port=42518
```

4.  Open `http://localhost:42518/` in your browser. You should see the default ColdBox app template.

5.  Open `/tests` in your browser. You should see the TestBox test browser.
    This is useful to find a specific test or group of tests to run _before_ running them.

6.  Open `/tests/runner.cfm` in your browser. You should see the TestBox test runner for our project.
    This is running all of our tests by default. We can create our own test runners as needed.

All your tests should be passing at this point. 😉

7.  Show routes file. Explain routing by convention.
8.  Show `Main.index`.
9.  Explain `event`, `rc`, and `prc`.
10. Explain views. Point out view conventions.
11. Briefly touch on layouts.
12. Show how `rc` and `prc` are used in the views.

13. Add an about page
    a. Add an `views/about/index.cfm`. It works!
    b. Add an `about` handler. Now it breaks! Talk about reinits.
    c. Add an `index` action. Back to working!

Reinits
What is cached?

*   Singletons

14. Copy in simple bootstrap theme and layout

```html
<cfoutput>
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <title>SoapBox</title>

        <base href="#event.getHTMLBaseURL()#" />

	    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
        <link rel="stylesheet" href="/includes/css/app.css">

        <script defer src="https://use.fontawesome.com/releases/v5.0.6/js/all.js"></script>
    </head>
    <body>
        <nav class="navbar navbar-expand-lg navbar-light bg-light fixed-top main-navbar">
            <a class="navbar-brand" href="#event.buildLink( url = "/" )#">
                <i class="fas fa-bullhorn mr-2"></i>
                SoapBox
            </a>
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="##navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
        </nav>

        <main role="main" class="container">
            #renderView()#
        </main>

        <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
    </body>
</html>
</cfoutput>
```

```css
/* includes/app.css */

/* For fixed navbar */
body {
    font-family: "Lato", sans-serif;

    min-height: 75rem;
    padding-top: 4.5rem;
}

.main-navbar {
    box-shadow: 0 15px 30px 0 rgba(0, 0, 0, 0.11), 0 5px 15px 0 rgba(0, 0, 0, 0.08);
}
```

15. Install [commandbox-migrations](https://www.forgebox.io/view/commandbox-migrations)

```sh
install commandbox-migrations
```

You should see a list of available commands with `migrate ?`.

16. Initalize migrations using `migrate init`

17. Change the grammar in your box.json to `MySQLGrammar`

(Bonus. Not needed since we are using `queryExecute` directly.)

```sh
package set cfmigrations.defaultGrammar=MySQLGrammar
```

17. Install [commandbox-dotenv](https://www.forgebox.io/view/commandbox-dotenv)

18. Create a `.env` file. Fill it in appropraitely. (We'll fill it in with our
    Docker credentials from before.)

```
DB_CLASS=org.gjt.mm.mysql.Driver
DB_CONNECTIONSTRING=jdbc:mysql://localhost:3306/soapbox?useUnicode=true\&characterEncoding=UTF-8\&useLegacyDatetimeCode=true
DB_USER=root
DB_PASSWORD=soapbox
```

19. Reload your shell in your project root. (`reload`)

20. Install cfmigrations using `migrate install`. (This will also test that you can connect to your database.)

21. Create a users migration

```sh
migrate create create_users_table
```

22. Fill in the migration.

```
component {

    function up( schema ) {
        queryExecute( "
            CREATE TABLE `users` (
                `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
                `username` VARCHAR(255) NOT NULL UNIQUE,
                `email` VARCHAR(255) NOT NULL UNIQUE,
                `password` VARCHAR(255) NOT NULL,
                `createdDate` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                `modifiedDate` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                CONSTRAINT `pk_users_id` PRIMARY KEY (`id`)
            )
        " );

        // schema.create( "users", function( table ) {
        //     table.increments( "id" );
        //     table.string( "username" ).unique();
        //     table.string( "email" ).unique();
        //     table.string( "password" );
        //     table.timestamp( "createdDate" );
        //     table.timestamp( "modifiedDate" );
        // } );
    }

    function down( schema ) {
        queryExecute( "DROP TABLE `users`" );

        // schema.drop( "users" );
    }

}
```

23. Run the migration up.

```sh
migrate up
```

QB will be optional

24. Install `qb`

```sh
install qb
```

25. Configure `qb`.

```js
// config/ColdBox.cfc
moduleSettings = {
    qb = {
        defaultGrammar = "MySQLGrammar"
    }
};
```

```js
// Application.cfc
variables.util = new coldbox.system.core.util.Util();

this.datasources = {
    "soapbox" = {
        "class" = util.getSystemSetting( "DB_CLASS" ),
        "connectionString" = util.getSystemSetting( "DB_CONNECTIONSTRING" ),
        "username" = util.getSystemSetting( "DB_USER" ),
        "password" = util.getSystemSetting( "DB_PASSWORD" )
    }
};
this.datasource = "soapbox";
```

26. Play around grabbing data from the database using `qb`.

```
// handlers/Main.cfc

// property name="query" inject="provider:QueryBuilder@qb";

function index( event, rc, prc ) {
    // prc.users = query.from( "users" ).get();
    prc.users = queryExecute( "SELECT * FROM users", {}, { returntype = "array" } );
    event.setView( "main/index" );
}
```

```html
// views/Main/index.cfm

<cfoutput>
    <cfdump var="#prc.users#" label="users" />
</cfoutput>
```

Insert data directly in to the database and show it returning.

27. Start register flow

28. Delete the MainBDDTest

29. Install `cfmigrations` as a dev dependency. `install cfmigrations --saveDev`

30. Configure `tests/Application.cfc`

```js
// tests/Application.cfc
variables.util = new coldbox.system.core.util.Util();

this.datasources = {
    "soapbox" = {
        "class" = util.getSystemSetting( "DB_CLASS" ),
        "connectionString" = util.getSystemSetting( "DB_CONNECTIONSTRING" ),
        "username" = util.getSystemSetting( "DB_USER" ),
        "password" = util.getSystemSetting( "DB_PASSWORD" )
    }
};
this.datasource = "soapbox";

function onRequestStart() {
    structDelete( application, "cbController" );
}
```

31. Create a `tests/resources/BaseIntegrationSpec.cfc`

```
component extends="coldbox.system.testing.BaseTestCase" {

    property name="migrationService" inject="MigrationService@cfmigrations";

    this.loadColdBox = true;
    this.unloadColdBox = false;

    function beforeAll() {
        super.beforeAll();
        application.wirebox.autowire( this );
        if ( ! request.keyExists( "migrationsRan" ) ) {
            migrationService.setMigrationsDirectory( "/root/resources/database/migrations" );
            migrationService.setDatasource( "soapbox" );
            migrationService.runAllMigrations( "down" );
            migrationService.runAllMigrations( "up" );
            request.migrationsRan = true;
        }
    }


    /**
     * @aroundEach
     */
    function wrapInTransaction( spec ) {
        transaction action="begin" {
            try {
                spec.body();
            }
            catch ( any e ) {
                rethrow;
            }
            finally {
                transaction action="rollback";
            }
        }
    }

}
```

32. Create a `tests/specs/integration/RegistrationSpec.cfc`

```
component extends="tests.resources.BaseIntegrationSpec" {

    function run() {
        describe( "registration", function() {
            it( "can register a user", function() {
                fail( "test not implemented yet" );
            } );
        } );
    }

}
```

33. Fill in the test

```
it( "can register a user", function() {
    expect( queryExecute( "select * from users", {}, { returntype = "array" } ) ).toBeEmpty();

    var event = post( "/registration", {
        "username" = "elpete",
        "email" = "eric@elpete.com",
        "password" = "mypass1234",
        "passwordConfirmation" = "mypass1234"
    } );

    expect( event.getValue( "relocate_URI", "" ) ).toBe( "/" );

    var users = query.from( "users" ).get();
    expect( users ).toBeArray();
    expect( users ).toHaveLength( 1 );
    expect( users[ 1 ].username ).toBe( "elpete" );
} );
```

34. Write the production code

```
// config/Routes.cfm

resources("registration");
```

```
// handlers/registration.cfc
component {

    property name="query" inject="provider:QueryBuilder@qb";

    function create( event, rc, prc ) {
        queryExecute( "
                INSERT INTO `users` ( `email`, `username`, `password` )
                VALUES ( ?, ?, ? )
            ",
            [ rc.email, rc.username, rc.password ]
        );

        relocate( uri = "/" );
    }

}
```

35. Create a route to populate this form

```js
function new( event, rc, prc ) {
    return event.setView( "registration/new" );
}
```

```html
<!-- views/registration/new.cfm -->
<cfoutput>
    <div class="card">
        <h4 class="card-header">Register for SoapBox</h4>
        <form class="form panel card-body" method="POST" action="#event.buildLink( "registration" )#">
            <div class="form-group">
                <label for="email" class="control-label">Email</label>
                <input id="email" name="email" type="email" class="form-control" placeholder="Email" />
            </div>
            <div class="form-group">
                <label for="username" class="control-label">Username</label>
                <input id="username" name="username" type="text" class="form-control" placeholder="Username" />
            </div>
            <div class="form-group">
                <label for="password" class="control-label">Password</label>
                <input id="password" name="password" type="password" class="form-control" placeholder="Password" />
            </div>
            <div class="form-group">
                <button type="submit" class="btn btn-primary">Register</button>
            </div>
        </form>
    </div>
</cfoutput>
```

36. Add a register link to the navbar

```html
<div class="collapse navbar-collapse" id="navbarSupportedContent">
    <ul class="navbar-nav ml-auto">
        <a href="#event.buildLink( "registration.new" )#" class="nav-link">Register</a>
    </ul>
</div>
```

Bonus points for tests first for next part

37. Add [BCyrpt](https://github.com/coldbox-modules/cbox-bcrypt)

```sh
install bcyrpt
```

38. Bcrypt the password

```
component {

    property name="bcrypt" inject="@BCrypt";

    function create( event, rc, prc ) {
        queryExecute( "
                INSERT INTO `users` ( `email`, `username`, `password` )
                VALUES ( ?, ?, ? )
            ",
            [ rc.email, rc.username, bcrypt.hashPassword( rc.password ) ]
        );

        relocate( uri = "/" );
    }
}
```

39. Try it again (will probably want a migrate fresh)

TODO: We probably want a WireBox intro somewhere. Just the basics. Breeze past most of it.

Add log in link

40. Create rants migrations

```
migrate create create_rants_table
```

```
component {

    function up( schema ) {
        queryExecute( "
            CREATE TABLE `rants` (
                `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
                `body` TEXT NOT NULL,
                `createdDate` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                `modifiedDate` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                `userId` INTEGER UNSIGNED NOT NULL,
                CONSTRAINT `pk_rants_id` PRIMARY KEY (`id`),
                CONSTRAINT `fk_rants_userId` FOREIGN KEY (`userId`) REFERENCES `users` (`id`) ON UPDATE CASCADE ON DELETE CASCADE
            )
        " );
    }

    function down( schema ) {
        queryExecute( "DROP TABLE `rants`" );
    }

}
```

41. Create a Rant object

```
// Rant.cfc
component accessors="true" {

    property name="UserService" inject="id";

    property name="id";
    property name="body";
    property name="createdDate";
    property name="modifiedDate";
    property name="userId";

    function getUser() {
        return userSerivce.retrieveUserById( getUserId() );
    }

}
```

42. Create RantService.cfc

```
component {

    function getAll() {
        return queryExecute(
            "SELECT * FROM `rants` ORDER BY `createdDate` DESC",
            [],
            { returntype = "array" }
        ).map( function ( rant ) {
            return populator.populateFromStruct(
                wirebox.getInstance( "Rant" ),
                rant
            );
        } );
    }

    function save( rant ) {
        rant.setModifiedDate( now() );
        queryExecute(
            "
                INSERT INTO `rants` (`body`, `updatedDate`, `userId`)
                VALUES (?, ?, ?)
            ",
            [
                rant.getBody(),
                { value = rant.getModifiedDate(), cfsqltype = "CF_SQL_TIMESTAMP" },
                rant.getUserId()
            ],
            { result = "local.result" }
        );
        rant.setId( result.GENERATED_KEY );
        return rant;
    }

}
```

43. Rants CRUD

```
// config/routes.cfm
resources( "rants" );
```

```
// handlers/rants.cfc
component {

    property name="RantService" inject="id";

    function index( event, rc, prc ) {
        prc.rants = rantService.getAll();
        event.setView( "rants/index" );
    }

    function new( event, rc, prc ) {
        event.setView( "rants/new" );
    }

    function create( event, rc, prc ) {
        rc.userId = auth().getUserId();
        var rant = populateModel( getInstance( "Rant" ) );
        rantService.save( rant );
        relocate( "rants" );
    }

}
```

```
// views/rants/index.cfm
<cfoutput>
    <cfif prc.rants.isEmpty()>
        <h3>No rants yet</h3>
        <a href="#event.buildLink( "rants.new" )#">Start one now!</a>
    <cfelse>
        <a href="#event.buildLink( "rants.new" )#" class="btn btn-link">Start a new rant!</a>
        <cfloop array="#prc.rants#" item="rant">
            <div class="card mb-3">
                <div class="card-header">
                    <strong>#rant.getUser().getUsername()#</strong> said at #dateTimeFormat( rant.getCreatedDate(), "h:nn:ss tt" )# on #dateFormat( rant.getCreatedDate(), "mmm d, yyyy")#
                </div>
                <div class="panel card-body">
                    #rant.getBody()#
                </div>
            </div>
        </cfloop>
    </cfif>
</cfoutput>
```

```
// views/rants/new.cfm
<cfoutput>
    <div class="card">
        <h4 class="card-header">Start a Rant</h4>
        <form class="form panel card-body" method="POST" action="#event.buildLink( "rants" )#">
            <div class="form-group">
                <textarea name="body" class="form-control" placeholder="What's on your mind?" rows="10"></textarea>
            </div>
            <div class="form-group">
                <button type="submit" class="btn btn-primary">Rant About It!</button>
            </div>
        </form>
    </div>
</cfoutput>
```

```
// layouts/Main.cfm
<div class="collapse navbar-collapse" id="navbarSupportedContent">
    <ul class="navbar-nav">
        <li><a href="#event.buildLink( "rants.new" )#" class="nav-link">Start a Rant</a></li>
    </ul>
    <ul class="navbar-nav ml-auto">
        <cfif auth().isLoggedIn()>
            <form method="POST" action="#event.buildLink( "logout" )#">
                <input type="hidden" name="_method" value="DELETE" />
                <button type="submit" class="btn btn-link nav-link">Log Out</button>
            </form>
        <cfelse>
            <a href="#event.buildLink( "registration.new" )#" class="nav-link">Register</a>
            <a href="#event.buildLink( "login" )#" class="nav-link">Log In</a>
        </cfif>
    </ul>
</div>
```
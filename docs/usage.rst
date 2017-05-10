Manual Usage
============

**Note:** The following page provides examples on how to configure and use
Sentry directly. It is **highly recommended** that you use one of the provided
integrations instead if possible.

Installation
------------

Using Maven:

.. sourcecode:: xml

    <dependency>
        <groupId>io.sentry</groupId>
        <artifactId>sentry</artifactId>
        <version>1.0.0</version>
    </dependency>

Using Gradle:

.. sourcecode:: groovy

    compile 'io.sentry:sentry:1.0.0'

Using SBT:

.. sourcecode:: scala

    libraryDependencies += "io.sentry" % "sentry" % "1.0.0"

For other dependency managers see the `central Maven repository <https://search.maven.org/#artifactdetails%7Cio.sentry%7Csentry%7C1.0.0%7Cjar>`_.

Capture an Error
----------------

To report an event manually you need to initialize a ``SentryClient``. It is recommended
that you use the static API via the ``Sentry`` class, but you can also construct and manage
your own ``SentryClient`` instance. An example of each style is shown below:

.. sourcecode:: java

    import io.sentry.context.Context;
    import io.sentry.event.BreadcrumbBuilder;
    import io.sentry.event.UserBuilder;

    public class MyClass {
        private static SentryClient sentry;

        public static void main(String... args) {
            /*
            It is recommended that you use the DSN detection system, which
            will check the environment variable "SENTRY_DSN" and the Java
            System Property "sentry.dsn". This makes it easier to provide
            and adjust your DSN without needing to change your code.
            */
            Sentry.init();

            // You can also manually provide the DSN to the ``init`` method.
            String dsn = args[0];
            Sentry.init(dsn);

            /*
            It is possible to go around the static ``Sentry`` API, which means
            you are responsible for making the SentryClient instance available
            to your code.
            */
            sentry = SentryClientFactory.sentryClient();

            MyClass myClass = new MyClass();
            myClass.logWithStaticAPI();
            myClass.logWithInstanceAPI();
        }

        /**
         * An example method that throws an exception.
         */
        void unsafeMethod() {
            throw new UnsupportedOperationException("You shouldn't call this!");
        }

        /**
         * Examples using the (recommended) static API.
         *
         * Note that the ``Sentry.init`` method must be called before the static API
         * is used, otherwise a ``NullPointerException`` will be thrown.
         */
        void logWithStaticAPI() {
            /*
            Record a breadcrumb in the current context which will be sent
            with the next event(s). By default the last 100 breadcrumbs are kept.
            */
            Sentry.record(new BreadcrumbBuilder().setMessage("User made an action").build());

            // Set the user in the current context.
            Sentry.setUser(new UserBuilder().setEmail("hello@sentry.io").build());

            /*
            This sends a simple event to Sentry using the statically stored instance
            that was created in the ``main`` method.
            */
            Sentry.capture("This is a test");

            try {
                unsafeMethod();
            } catch (Exception e) {
                // This sends an exception event to Sentry using the statically stored instance
                // that was created in the ``main`` method.
                Sentry.capture(e);
            }
        }

        /**
         * Examples that use the SentryClient instance directly.
         */
        void logWithInstanceAPI() {
            // Retrieve the current context.
            Context context = sentry.getContext();

            /*
            Record a breadcrumb in the current context which will be sent
            with the next event(s). By default the last 100 breadcrumbs are kept.
            */
            context.recordBreadcrumb(new BreadcrumbBuilder().setMessage("User made an action").build());

            // Set the user in the current context.
            context.setUser(new UserBuilder().setEmail("hello@sentry.io").build());

            // This sends a simple event to Sentry.
            sentry.sendMessage("This is a test");

            try {
                unsafeMethod();
            } catch (Exception e) {
                // This sends an exception event to Sentry.
                sentry.sendException(e);
            }
        }
    }

Building More Complex Events
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For more complex messages, you'll need to build an ``Event`` with the
``EventBuilder`` class:

.. sourcecode:: java

    import io.sentry.Sentry;
    import io.sentry.event.Event;
    import io.sentry.event.EventBuilder;
    import io.sentry.event.interfaces.ExceptionInterface;

    public class MyClass {
        public static void main(String... args) {
            Sentry.init();
        }

        void unsafeMethod() {
            throw new UnsupportedOperationException("You shouldn't call this!");
        }

        void logSimpleMessage() {
            // This sends an event to Sentry.
            EventBuilder eventBuilder = new EventBuilder()
                            .withMessage("This is a test")
                            .withLevel(Event.Level.INFO)
                            .withLogger(MyClass.class.getName());
            Sentry.capture(eventBuilder);
        }

        void logException() {
            try {
                unsafeMethod();
            } catch (Exception e) {
                // This sends an exception event to Sentry.
                EventBuilder eventBuilder = new EventBuilder()
                                .withMessage("Exception caught")
                                .withLevel(Event.Level.ERROR)
                                .withLogger(MyClass.class.getName())
                                .withSentryInterface(new ExceptionInterface(e));
                Sentry.capture(eventBuilder);
            }
        }
 }

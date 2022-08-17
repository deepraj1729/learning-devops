# NGROK

<p align="center">
  <img src="Assets/ngrok.png" />
</p>

## What is Ngrok?
Ngrok is software that allows you to run web servers through the cloud. The traffic is relayed through your own computer and provides a secure HTTP URL to tunnel to. No need for port forwarding! You can easily build RESTful APIs with the tunneling service. The service is free too, if you don’t have a registered account then your Ngrok server will run for 2 hours. You can choose to have a paid plan so that it runs 24/7 as well.

## Getting started with Ngrok:
First, you'll need some sort of web service running on your machine. It should be available at http://localhost:[any port]. If you already have one, you can skip to Step 2. If not, we'll set one up using Python SimpleHTTPServer (ngrok actually has a built in file server but let's not worry about that now).

[Read More](https://ngrok.com/docs/getting-started)

## FastAPI Implementation:
Install the python wrapper for ngrok using pip:

    pip install pyngrok

In server.py, where our FastAPI app is initialized, we should add a variable that let’s us configure from an environment variable whether we want to tunnel to localhost with ngrok. We can initialize the pyngrok tunnel in this same place.

        ```python
        import os
        import sys

        from fastapi import FastAPI
        from fastapi.logger import logger
        from pydantic import BaseSettings


        class Settings(BaseSettings):
            # ... The rest of our FastAPI settings

            BASE_URL = "http://localhost:8000"
            USE_NGROK = os.environ.get("USE_NGROK", "False") == "True"


        settings = Settings()


        def init_webhooks(base_url):
            # Update inbound traffic via APIs to use the public-facing ngrok URL
            pass


        # Initialize the FastAPI app for a simple web server
        app = FastAPI()

        if settings.USE_NGROK:
            # pyngrok should only ever be installed or initialized in a dev environment when this flag is set
            from pyngrok import ngrok

            # Get the dev server port (defaults to 8000 for Uvicorn, can be overridden with `--port`
            # when starting the server
            port = sys.argv[sys.argv.index("--port") + 1] if "--port" in sys.argv else 8000

            # Open a ngrok tunnel to the dev server
            public_url = ngrok.connect(port).public_url
            logger.info("ngrok tunnel \"{}\" -> \"http://127.0.0.1:{}\"".format(public_url, port))

            # Update any base URLs or webhooks to use the public ngrok URL
            settings.BASE_URL = public_url
            init_webhooks(public_url)

        # ... Initialize routers and the rest of our app
        ```

    Now FastAPI can be started by the usual means, with Uvicorn, setting USE_NGROK to open a tunnel.

        USE_NGROK=True uvicorn server:app


    ## End-to-End Testing:
    Some testing use-cases might mean we want to temporarily expose a route via a pyngrok tunnel to fully validate a workflow. For example, an internal end-to-end tester, a step in a pre-deployment validation pipeline, or a service that automatically updates a status page.

    Whatever the case may be, extending unittest.TestCase and adding our own fixtures that start the dev server and open a pyngrok tunnel is relatively simple. This snippet builds on the Flask example above, but it could be easily modified to work with Django or another framework if its dev server was started/stopped in the start_dev_server() and stop_dev_server() methods and PORT was changed.

            ```python
            import unittest
            import threading

            from flask import request
            from pyngrok import ngrok
            from urllib import request

            from server import create_app


            class PyngrokTestCase(unittest.TestCase):
                # Default Flask port
                PORT = 5000

                @classmethod
                def start_dev_server(cls):
                    app = create_app()

                    def shutdown():
                        request.environ.get("werkzeug.server.shutdown")()

                    @app.route("/shutdown", methods=["POST"])
                    def route_shutdown():
                        shutdown()
                        return "", 204

                    threading.Thread(target=app.run).start()

                @classmethod
                def stop_dev_server(cls):
                    req = request.Request("http://localhost:5000/shutdown", method="POST")
                    request.urlopen(req)

                @classmethod
                def init_webhooks(cls, base_url):
                    webhook_url = "{}/foo".format(base_url)

                    # ... Update inbound traffic via APIs to use the public-facing ngrok URL

                @classmethod
                def init_pyngrok(cls):
                    # Open a ngrok tunnel to the dev server
                    public_url = ngrok.connect(PORT).public_url

                    # Update any base URLs or webhooks to use the public ngrok URL
                    cls.init_webhooks(public_url)

                @classmethod
                def setUpClass(cls):
                    cls.start_dev_server()

                    cls.init_pyngrok()

                @classmethod
                def tearDownClass(cls):
                    cls.stop_dev_server()
            ```
## References:
- [Getting started with Ngrok](https://ngrok.com/docs/getting-started)
- [pyngrok Documentation](https://pyngrok.readthedocs.io/en/latest/integrations.html#fastapi)
# django-serfa

`django-serfa` is an experimental package for declaring the services a Django test suite needs and starting them with [Testcontainers for Python](https://testcontainers-python.readthedocs.io/). It is inspired by [django-rdtwt](https://github.com/wonkybream/django-rdtwt), but aims to support generic services rather than only a database.

## Current status

The runner validates `SERFA_DEFINITION`, starts the containers before the tests, and stops them during test environment teardown. Integration is still incomplete: it does not explicitly wait for services to become ready or automatically configure Django's `DATABASES` or other settings with container addresses. Host ports are fixed and must be available.

## Installation and configuration

You need Python 3.12 or newer (but earlier than Python 4), Docker accessible to the test process, and [uv](https://docs.astral.sh/uv/). To install this repository into another uv project while developing locally:

```console
uv add --editable /path/to/django-serfa
```

In your Django test settings:

```python
TEST_RUNNER = "django_serfa.runner.ServiceFactoryRunner"

SERFA_DEFINITION = {
    "services": [
        {
            "image": "redis:7-alpine",
            "name": "redis-test",
            "ports": [{"container": 6379, "host": 16379}],
        },
    ],
}
```

Each service requires `image` and `ports` (which may be an empty list). `name` is optional. The schema also accepts `network`, although the runner does not apply it yet. If your application needs the service in this example, configure its client to use `localhost:16379` in the test settings.

Run the test suite with Django's usual command:

```console
uv run python manage.py test
```

## Development

From this repository:

```console
uv sync
uv run python runtests.py
```

`uv.lock` pins the resolved dependency versions and should be committed. The current tests use Docker and require access to its daemon. Some test cases and settings mutations also need revision before the suite can reliably verify the integration.

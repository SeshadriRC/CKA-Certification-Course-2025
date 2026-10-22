Here are two simple Python Dockerfiles you can explain confidently in a DevOps interview.

## 1. Simple Dockerfile — Python

Project structure:

```
python-app/
├── app.py
└── Dockerfile
```

app.py

```
print("Hello from Python Docker container!")
```

Dockerfile

```

FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]

```

Build and run:

```
docker build -t python-app .
docker run --rm python-app
```

Output:

```
Hello from Python Docker container!
```

How to explain in an interview:

- `FROM` — uses the Python base image.
- `WORKDIR` — sets the working directory inside the container.
- `COPY` — copies the Python application into the image.
- `CMD` — specifies the default command when the container starts.

## 2. Multistage Dockerfile — Python

For interview purposes, we'll use a requirements file to install a Python dependency in the builder stage and copy the installed dependencies into the final image.

Project structure:

```
python-app/
├── app.py
├── requirements.txt
└── Dockerfile
```

app.py

```
import requestsresponse = requests.get("https://example.com", timeout=10)print("Status code:", response.status_code)
```

requirements.txt

```
requests
```

Dockerfile

```

# Stage 1: Builder
FROM python:3.12-slim AS builder

WORKDIR /app

COPY requirements.txt .

# Install Python dependencies from requirements.txt into /install without caching downloaded packages
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Stage 2: Final image
FROM python:3.12-slim

WORKDIR /app

# Copy the installed Python dependencies from the builder stage to the final image
COPY --from=builder /install /usr/local

COPY app.py .

CMD ["python", "app.py"]

```

**final struture**

```bash
/
├── app/
│   └── app.py
└── usr/
    └── local/
        └── lib/
            └── python3.12/
                └── site-packages/
                    └── requests/
```

Build and run:

```
docker build -t python-multistage .
docker run --rm python-multistage
```

The app makes an HTTP request, so it needs network access when run.

### How multistage works

Stage 1 — Builder

Install Python dependencies into `/install`.

Copy only the installed dependencies

Stage 2 — Final image

Copy dependencies and application code into a clean Python image.

The key instruction is:

```
COPY --from=builder /install /usr/local
```

It copies files from the `builder` stage into the final stage.

### Interview difference

| Simple Dockerfile               | Multistage Dockerfile                            |
| ------------------------------- | ------------------------------------------------ |
| Usually one build stage         | Uses multiple stages                             |
| Simple to write and maintain    | Separates building from runtime                  |
| Suitable for small applications | Useful when build tools aren't needed at runtime |
| May include build dependencies  | Can exclude unnecessary build tools              |

Interview answer: “I use a multistage Docker build to separate the build environment from the runtime image. This helps reduce the final image size and attack surface by excluding unnecessary build tools.”

One important detail: multistage builds do not automatically make every image smaller. The benefit depends on what you leave out of the final stage.

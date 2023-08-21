ARG PYTHON_VERSION=3.11.4
FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/openfaas/classic-watchdog:0.2.1 as watchdog
FROM --platform=${TARGETPLATFORM:-linux/amd64} python:${PYTHON_VERSION}-alpine as base

ARG TARGETPLATFORM
ARG BUILDPLATFORM

ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8
ENV PYTHONDONTWRITEBYTECODE 0 # ensure pyc files are used for performance
ENV PYTHONFAULTHANDLER 0 # disable Python tracebacks on segfault

RUN apk update && apk upgrade -y

# Allows you to add additional packages via build-arg
ARG ADDITIONAL_PACKAGE
RUN apk --no-cache add ca-certificates ${ADDITIONAL_PACKAGE}


FROM base AS depbuilder


RUN apk add -y --no-install-recommends gcc
RUN pip install --upgrade pip
RUN pip install pipenv

WORKDIR /home/app/

COPY function/Pipfile
COPY function/Pipfile.lock
RUN PIPENV_VENV_IN_PROJECT=1 pipenv install --python=${PYTHON_VERSION} --deploy


FROM base AS runtime

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

WORKDIR /home/app/
COPY index.py
COPY function
COPY --from=python-depbuilder /home/app/.venv

# Add non root user
RUN addgroup -S app && adduser app -S -G app
RUN chown -R app /home/app

USER root

# Allow any user-id for OpenShift users.
RUN chown -R app:app ./ && \
  chmod -R 777 /home/app/.venv

USER app

ENV PATH="/home/app/.venv/bin:$PATH"
ENV PYTHONPATH=$PYTHONPATH:/home/app/.venv

ENV fprocess="python3 index.py"
EXPOSE 8080

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]

ARG PYTHON_VERSION=3.11.4
FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/openfaas/classic-watchdog:0.2.1 as watchdog
FROM --platform=${TARGETPLATFORM:-linux/amd64} python:${PYTHON_VERSION}-alpine as base

ARG TARGETPLATFORM
ARG BUILDPLATFORM

ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8
ENV PYTHONDONTWRITEBYTECODE 0 # ensure pyc files are used for performance
ENV PYTHONFAULTHANDLER 0 # disable Python tracebacks on segfault

RUN apk update && apk upgrade

# Allows you to add additional packages via build-arg
ARG ADDITIONAL_PACKAGE
RUN apk --no-cache add ca-certificates ${ADDITIONAL_PACKAGE}


FROM base AS depbuilder

RUN apk add gcc
RUN pip install pipenv

WORKDIR /home/app/

COPY function/Pipfile /home/app
COPY function/Pipfile.lock /home/app
RUN PIPENV_VENV_IN_PROJECT=1 pipenv install --python=${PYTHON_VERSION} --deploy


FROM base AS runtime

ARG SOURCE_REPO
LABEL org.opencontainers.image.source ${SOURCE_REPO}

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

WORKDIR /home/app/
COPY index.py index.py
COPY function function
COPY --from=depbuilder /home/app/.venv /home/app/.venv

# Add non root user
RUN addgroup -S app && adduser app -S -G app --uid 12000
RUN chown -R app /home/app

USER app

ENV PATH="/home/app/.venv/bin:$PATH"
ENV PYTHONPATH=$PYTHONPATH:/home/app/.venv
ARG PYTHON_BOOTSTRAP
RUN python3 -c "${PYTHON_BOOTSTRAP}"

ENV fprocess="python3 index.py"
EXPOSE 8080

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]

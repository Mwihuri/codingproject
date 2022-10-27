#################
# Build stage
#################
FROM registry.ci.edgehawk-security.com/ci/sphinx-readthedocs as build

WORKDIR /app

COPY ./ ./
RUN pip3 install sphinx-markdown-tables
RUN pip3 install recommonmark
RUN sphinx-multiversion . _build/html

#################
# Release stage
#################
FROM registry.ci.edgehawk-security.com/cache/library/nginx:1.21.4-alpine as release

# Arguments
ARG COMMIT_SHA
ARG VERSION

# Labels
LABEL COMMIT_SHA=${COMMIT_SHA}

# Environment Variables
ENV VERSION=${VERSION}

# Copy files
COPY --from=build /app/_build/html/ /usr/share/nginx/html/
COPY ./nginx.conf /etc/nginx/conf.d/default.conf

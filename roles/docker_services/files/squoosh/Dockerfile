FROM node:24-alpine AS build

# renovate: datasource=git-refs depName=https://github.com/GoogleChromeLabs/squoosh currentValue=dev
ARG SQUOOSH_COMMIT=e8d35e0fb66eb16eff6fe8fc773eabcbb7128de3

WORKDIR /src
RUN apk add --no-cache git \
    && git clone https://github.com/GoogleChromeLabs/squoosh.git . \
    && git checkout "${SQUOOSH_COMMIT}"
RUN npm ci
RUN npm run build

FROM nginx:1.31.2-alpine

COPY --from=build /src/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

# Builder image
FROM rust:latest AS builder
# Install build dependencies
# RUN dnf install -y rust cargo openssl-devel
# Create new rust binary
RUN USER=root cargo new --bin airdrome_api
# Set working directory
WORKDIR ${pwd}/airdrome_api
# Add dependencies file
COPY ./Cargo.toml ./Cargo.toml
# Build dependencies
RUN cargo build --release
# Clean up
RUN rm src/*rs
# Add app source files
ADD ./src ./src
ADD ./migrations ./migrations
ADD ./Cargo.lock ./Cargo.lock
ADD ./Cargo.toml ./Cargo.toml
ADD ./sqlx-data.json ./sqlx-data.json
# Clean up build from dependencies
RUN rm ./target/release/deps/airdrome_api*
# Unset the database URL for offline query verification
RUN unset DATABASE_URL
# Build app binary
RUN cargo build --release

# App image
FROM fedora:latest
# Expose server port
EXPOSE 8080
# User to run app
ENV APP_USER=rust
# Set directory of app
ARG APP=/srv/airdrome_api
# Update
# RUN dnf update -y \
# && dnf clean all
# Create user and group to run app
RUN groupadd $APP_USER \
    && useradd -g $APP_USER $APP_USER \
    && mkdir -p $APP
# Copy binary from builder
COPY --from=builder /airdrome_api/target/release/airdrome_api $APP/airdrome_api
# Change ownership of app binary
RUN chown -R $APP_USER:$APP_USER $APP
# Set current user
USER $APP_USER
# Set working directory
WORKDIR $APP

# Start app
CMD ["./airdrome_api"]

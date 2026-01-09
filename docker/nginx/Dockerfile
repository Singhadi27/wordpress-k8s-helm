FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    libpcre3 \
    libpcre3-dev \
    zlib1g \
    zlib1g-dev \
    libssl-dev \
    perl \
    wget \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Download OpenResty
WORKDIR /tmp
RUN wget https://openresty.org/download/openresty-1.21.4.1.tar.gz \
    && tar -xzf openresty-1.21.4.1.tar.gz

# Build OpenResty with required options
WORKDIR /tmp/openresty-1.21.4.1
RUN ./configure --prefix=/opt/openresty \
    --with-pcre-jit \
    --with-ipv6 \
    --without-http_redis2_module \
    --with-http_iconv_module \
    --with-http_postgres_module \
    -j8 \
    && make \
    && make install

# Create nginx user
RUN useradd -r nginx

# Copy nginx configuration (we’ll add this later)
RUN mkdir -p /opt/openresty/nginx/conf

# Expose HTTP port
EXPOSE 80

# Start OpenResty
CMD ["/opt/openresty/nginx/sbin/nginx", "-g", "daemon off;"]


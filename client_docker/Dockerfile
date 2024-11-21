# Use Alpine Linux as the base image
FROM alpine:latest

# Install necessary packages
RUN apk add --no-cache wget ca-certificates

# Set environment variables for Clash version and architecture
# ENV CLASH_VERSION v1.18.0
# ENV CLASH_ARCH 386
# ENV CLASH_URL https://github.com/Kuingsmile/clash-core/releases/download/1.18/clash-linux-${CLASH_ARCH}-${CLASH_VERSION}.gz

# Create the configuration directory for Clash
RUN mkdir -p /root/.config/clash/

# Download and install Clash
# RUN wget ${CLASH_URL} -O /tmp/clash.gz \
#     && gunzip /tmp/clash.gz \
#     && chmod +x /tmp/clash \
#     && mv /tmp/clash /usr/local/bin/clash

# Copy and install Clash
COPY ./clash-linux-386-v1.18.0.gz /tmp/clash.gz
RUN gunzip /tmp/clash.gz \
    && chmod +x /tmp/clash \
    && mv /tmp/clash /usr/local/bin/clash

# Set the working directory
WORKDIR /root/.config/clash/

# Expose ports (if necessary)
EXPOSE 7890 7891 9090

# Run Clash with the specified configuration file
CMD ["clash", "-f", "/root/.config/clash/config.yaml"]

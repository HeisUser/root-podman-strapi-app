# Base image: Use Node.js 18 with Alpine Linux 3.18 as the foundation
# - node:18-alpine3.18 is a lightweight image with Node.js 18, suitable for Strapi v5 (supports Node 18–22)
# - Alpine is chosen for its small size, but requires additional packages for native dependencies
FROM node:18-alpine3.18

# Install dependencies required for sharp and other native modules
# - apk update: Updates the Alpine package index to ensure we get the latest packages
# - apk add --no-cache: Installs packages without caching them locally (keeps image size small)
# - build-base: Includes gcc, g++, and make for compiling native code
# - gcc, autoconf, automake: Additional tools for building native dependencies
# - zlib-dev, libpng-dev: Libraries for image processing (used by sharp)
# - nasm: Assembler for optimizing image libraries like libpng
# - bash: Adds bash shell for scripting (not strictly necessary but useful)
# - vips-dev: Development files for libvips, a fast image processing library used by sharp
# - git: Allows npm to fetch dependencies from Git repositories if needed
RUN apk update && apk add --no-cache build-base gcc autoconf automake zlib-dev libpng-dev nasm bash vips-dev git
# Installing libvips-dev for sharp Compatibility

# Define a build argument for the Node environment
# - ARG NODE_ENV=development: Sets a default value of "development" for the NODE_ENV variable
# - This can be overridden during build with --build-arg NODE_ENV=production
ARG NODE_ENV=development

# Set the NODE_ENV environment variable in the container
# - ENV NODE_ENV=${NODE_ENV}: Makes NODE_ENV available to all subsequent commands
# - In development mode, Strapi includes more debugging info; in production, it optimizes
ENV NODE_ENV=${NODE_ENV}

# Set the working directory to /opt/
# - WORKDIR /opt/: Changes the current directory for subsequent commands to /opt/
# - This is where dependencies will be installed initially
WORKDIR /opt/

# Copy package.json and package-lock.json (if it exists) to the working directory
# - COPY package*.json ./: Copies package files to /opt/
# - Using package*.json ensures both package.json and package-lock.json (if present) are copied
# - This step is separated to leverage Docker caching: if package files don’t change, npm install is cached
COPY package*.json ./

# Install node-gyp globally
# - RUN npm install -g node-gyp: Installs node-gyp, a tool for compiling native Node.js addons
# - Needed for dependencies like sharp that require native compilation
RUN npm install -g node-gyp

# Configure npm to increase fetch retry timeout and install dependencies
# - npm config set fetch-retry-maxtimeout 600000 -g: Sets a 10-minute timeout for npm fetches
# - This prevents failures due to slow network or large dependency downloads
# - npm install: Installs all dependencies listed in package.json into /opt/node_modules/
RUN npm config set fetch-retry-maxtimeout 600000 -g && npm install

# Update the PATH environment variable
# - ENV PATH=/opt/node_modules/.bin:$PATH: Adds /opt/node_modules/.bin to the PATH
# - This allows running executables (like strapi) installed by npm without specifying full paths
ENV PATH=/opt/node_modules/.bin:$PATH

# Change the working directory to /opt/app/
# - WORKDIR /opt/app/: Sets the directory where the app code will live
# - This separates the app source code from the node_modules installed in /opt/
WORKDIR /opt/app/

# Copy the entire project directory into the container
# - COPY ./ .: Copies all files from the host’s build context (project root) to /opt/app/
# - This includes src/, config/, public/, .env, etc., unless excluded by .dockerignore
COPY ./ .

# Change ownership of /opt/app/ to the node user
# - RUN chown -R node:node /opt/app/: Gives the non-root "node" user ownership of the app directory
# - This improves security by avoiding running Strapi as root
RUN chown -R node:node /opt/app/

# Switch to the node user
# - USER node: Sets all subsequent commands to run as the "node" user instead of root
# - Ensures Strapi runs with reduced privileges
USER node

## RESOLVED FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory
# ENV GENERATE_SOURCEMAP=false
# ENV NODE_OPTIONS=--max_old_space_size=4096
# Disable source map generation to reduce memory usage
# - ENV GENERATE_SOURCEMAP=false: Prevents Webpack (used by Strapi) from generating source maps
# - Source maps are useful for debugging but increase memory and build time; disabling helps avoid heap errors
ENV GENERATE_SOURCEMAP=false

# Increase Node.js heap size to prevent memory errors
# - ENV NODE_OPTIONS=--max_old_space_size=4096: Sets the JavaScript heap limit to 4GB
# - Addresses the "FATAL ERROR: Reached heap limit" by allowing more memory for strapi build
ENV NODE_OPTIONS=--max_old_space_size=4096

# Build the Strapi application
# - RUN ["npm", "run", "build"]: Executes the "build" script from package.json (typically "strapi build")
# - This compiles the admin panel and prepares the app for production
# - The NODE_OPTIONS above ensures it has enough memory to complete
RUN ["npm", "run", "build"]

# Expose port 1337 for Strapi
# - EXPOSE 1337: Documents that the container listens on port 1337 (Strapi’s default)
# - Doesn’t actually publish the port; that’s done with -p in podman run
EXPOSE 1337

# Define the default command to run the app
# - CMD ["npm", "run", "develop"]: Runs the "develop" script from package.json (typically "strapi develop")
# - Starts Strapi in development mode with auto-reloading; use "start" for production
CMD ["npm", "run", "develop"]
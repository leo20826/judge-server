FROM vnoj/judge-tier3:latest

ARG TAG=master

# 2. Limpieza de seguridad y herramientas básicas
RUN rm -f /etc/apt/sources.list.d/*.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        openjdk-25-jdk-headless \
        openjdk-25-jre-headless \
        golang \
        unzip \
        jq \
        ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# 3. Instalación de Rust, Kotlin y PyPy (CORREGIDO)
RUN ARCH=$([ $(uname -m) = "x86_64" ] && echo "amd64" || echo "arm64") && \
    curl --location -o scratch-run.zip $(curl -s https://api.github.com/repos/VNOI-Admin/scratch-run/releases/latest | grep -o -m 1 "https://github\.com.*.*linux_$ARCH\.zip") && \
        unzip scratch-run.zip && mv scratch-run /usr/bin/scratch-run && rm scratch-run.zip && \
    if [ "$(arch)" = x86_64 ]; then PYPY_ARCH=linux64; else PYPY_ARCH="$(arch)"; fi && \
    mkdir -p /opt/pypy2 && curl -L "$(curl https://pypy.org/download.html | grep "/pypy2.*$PYPY_ARCH" | head -n1 | cut -d'"' -f4)" | \
        tar xj -C /opt/pypy2 --strip-components=1 && \
    mkdir -p /opt/pypy3 && curl -L "$(curl https://pypy.org/download.html | grep "/pypy3.*$PYPY_ARCH" | head -n1 | cut -d'"' -f4)" | \
        tar xj -C /opt/pypy3 --strip-components=1 && \
    curl -L -okotlin.zip "$(curl -s https://api.github.com/repos/JetBrains/kotlin/releases | \
        jq -r '[.[] | select(.prerelease | not) | .assets | flatten | .[] | select((.name | startswith("kotlin-compiler")) and (.name | endswith(".zip"))) | .browser_download_url][0]')" && \
        unzip kotlin.zip && rm -rf /opt/kotlin && mv kotlinc /opt/kotlin && rm kotlin.zip && \
    # --- LIMPIEZA DE RUST PARA EVITAR ERROR ---
    rm -rf /home/judge/.rustup /home/judge/.cargo && \
    runuser judge -c 'curl https://sh.rustup.rs -sSf | sh -s -- -y'

ENV PATH="/opt/kotlin/bin:/opt/pypy2/bin:/opt/pypy3/bin:/home/judge/.cargo/bin:${PATH}"

# 4. INSTALACIÓN DEL SOFTWARE DEL JUDGE (Esto es lo que faltaba)
RUN mkdir -p /judge /problems && cd /judge && \
    curl -L https://github.com/VNOI-Admin/judge-server/archive/"${TAG}".tar.gz | tar -xz --strip-components=1 && \
    python3 -m venv --prompt=DMOJ /env && \
    /env/bin/pip3 install cython setuptools && \
    /env/bin/pip3 install -e . && \
    /env/bin/python3 setup.py develop && \
    HOME=~judge . ~judge/.profile && \
    runuser -u judge -w PATH -- /env/bin/dmoj-autoconf -V > /judge-runtime-paths.yml && \
    /judge/.docker/download_testlib_and_precompile

ENTRYPOINT ["/usr/bin/tini", "--", "/judge/.docker/entry"]

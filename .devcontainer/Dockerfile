FROM mcr.microsoft.com/devcontainers/base:debian

# Rename vscode user to dev (base image has vscode with UID 1000)
RUN usermod -l dev -d /home/dev -m vscode \
    && groupmod -n dev vscode \
    && sed -i 's/vscode/dev/g' /etc/sudoers.d/vscode || true \
    && mv /etc/sudoers.d/vscode /etc/sudoers.d/dev 2>/dev/null || true

# Install build tools and dependencies (matching GitHub Actions workflow)
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Build tools
    build-essential \
    cmake \
    pkg-config \
    gfortran \
    # Qt5 development
    qtbase5-dev \
    qtmultimedia5-dev \
    libqt5serialport5-dev \
    qttools5-dev \
    qttools5-dev-tools \
    # Libraries
    libfftw3-dev \
    libboost-all-dev \
    libportaudio2 \
    portaudio19-dev \
    libusb-1.0-0-dev \
    # Hamlib (radio control)
    libhamlib-dev \
    libhamlib-utils \
    # System libraries
    libudev-dev \
    # Additional tools
    git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# SSH quiet config
RUN mkdir -p /home/dev/.ssh \
    && printf 'Host *\n    StrictHostKeyChecking accept-new\n    LogLevel ERROR\n' > /home/dev/.ssh/config \
    && chmod 700 /home/dev/.ssh \
    && chmod 600 /home/dev/.ssh/config \
    && chown -R dev:dev /home/dev/.ssh

USER dev

# Set default working directory for SSH logins
WORKDIR /workspaces/wsjt-x-mcp
RUN echo 'cd /workspaces/wsjt-x-mcp 2>/dev/null' >> /home/dev/.bashrc

# tmux — terminal multiplexer for SSH sessions
USER root
RUN apt-get update && apt-get install -y --no-install-recommends tmux \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
USER dev

# Per-user npm prefix on PATH so claude (installed into ~/.npm-global/bin by
# postCreate) and the superclaude alias resolve. Set as image ENV, not appended
# to .bashrc: Debian's .bashrc returns early for non-interactive shells, so a
# PATH export there is invisible to `ssh <host> claude ...`, cron and scripts.
# ENV applies to every shell, interactive or not.
ENV PATH="/home/dev/.npm-global/bin:${PATH}"

# Force UTF-8 locale so tmux renders Unicode block glyphs (claude's splash logo)
# correctly. Must also come before the tmux guard.
RUN echo 'export LANG=C.UTF-8' >> /home/dev/.bashrc

# Persistent claude session: interactive SSH logins auto-attach a tmux session
# "main" running claude, so it survives disconnects (reconnect = continue).
RUN printf '%s\n' \
    '' \
    '# --- Persistent claude session (auto-attach tmux on interactive SSH) ---' \
    'if [ -n "$SSH_CONNECTION" ] && [ -z "$TMUX" ] && [ "$TERM_PROGRAM" != "vscode" ] && [ ! -f "$HOME/.no-tmux" ] && command -v tmux >/dev/null 2>&1; then' \
    '    case "$-" in' \
    '        *i*) exec tmux new-session -A -s main -c /workspaces/wsjt-x-mcp "claude --dangerously-skip-permissions; exec bash" ;;' \
    '    esac' \
    'fi' \
    >> /home/dev/.bashrc

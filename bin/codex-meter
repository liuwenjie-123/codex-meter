#!/usr/bin/env node

import { spawnSync } from "node:child_process";
import fs from "node:fs";
import { dirname, join } from "node:path";
import { platform as osPlatform, arch as osArch } from "node:os";
import { fileURLToPath } from "node:url";

const CLI_NAME = "codex-meter";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

function run(target) {
  const result = spawnSync(target, process.argv.slice(2), {
    stdio: "inherit",
  });
  if (result.error) {
    console.error(result.error.message);
    process.exit(1);
  }
  const code = typeof result.status === "number" ? result.status : 0;
  process.exit(code);
}

function runCommand(command, args) {
  const result = spawnSync(command, args, {
    stdio: "inherit",
  });
  if (result.error) {
    return false;
  }
  const code = typeof result.status === "number" ? result.status : 0;
  process.exit(code);
}

function findBun() {
  const exe = osPlatform() === "win32" ? "bun.exe" : "bun";
  const candidates = [
    "bun",
    process.env.BUN_INSTALL ? join(process.env.BUN_INSTALL, "bin", exe) : undefined,
    process.env.USERPROFILE ? join(process.env.USERPROFILE, ".bun", "bin", exe) : undefined,
    process.env.HOME ? join(process.env.HOME, ".bun", "bin", exe) : undefined,
  ].filter(Boolean);

  for (const candidate of candidates) {
    if (candidate === "bun" || fs.existsSync(candidate)) {
      const result = spawnSync(candidate, ["--version"], { stdio: "ignore" });
      if (!result.error && result.status === 0) {
        return candidate;
      }
    }
  }
}

const platformMap = {
  darwin: "darwin",
  linux: "linux",
  win32: "windows",
};
const archMap = {
  x64: "x64",
  arm64: "arm64",
};

let platform = platformMap[osPlatform()];
if (!platform) {
  platform = osPlatform();
}
let arch = archMap[osArch()];
if (!arch) {
  arch = osArch();
}

const base = `${CLI_NAME}-${platform}-${arch}`;
const binary = platform === "windows" ? `${CLI_NAME}.exe` : CLI_NAME;

function findBinary(startDir) {
  let current = startDir;
  for (;;) {
    const modules = join(current, "node_modules");
    if (fs.existsSync(modules)) {
      const entries = fs.readdirSync(modules);
      for (const entry of entries) {
        if (!entry.startsWith(base)) {
          continue;
        }
        const candidate = join(modules, entry, "bin", binary);
        if (fs.existsSync(candidate)) {
          return candidate;
        }
      }
    }
    const parent = dirname(current);
    if (parent === current) {
      return;
    }
    current = parent;
  }
}

const resolved = findBinary(__dirname);
if (!resolved) {
  const sourceEntry = join(__dirname, "..", "src", "index.ts");
  const bun = findBun();
  if (bun && fs.existsSync(sourceEntry) && runCommand(bun, [sourceEntry, ...process.argv.slice(2)])) {
    process.exit(0);
  }

  console.error(
    `It seems that your package manager failed to install the right version of ${CLI_NAME} CLI for your platform. You can try manually installing "${base}" package`
  );
  process.exit(1);
}

run(resolved);

<!--
  HASHNODE PUBLISH INSTRUCTIONS
  ══════════════════════════════════════════════════════
  1. Go to hashnode.com → Write → "Import article"
     OR paste content below into the Hashnode editor directly.

  2. After import / paste, fill in the UI fields:
     • Title       : OverlayFS Explained: How Docker's Filesystem Layers Actually Work
     • Subtitle    : 
     • Slug        : overlayfs-explained-how-docker-s-filesystem-layers-actually-work
     • Tags        : docker, filesystem, containers, overlayfs, copy-on-write
     • Cover image : https://calligra.dev/images/blog/overlayfs-explained-how-docker-s-filesystem-layers-actually-work/cover.png

  3. Under "SEO" settings:
     • Canonical URL: https://calligra.dev/blog/overlayfs-explained-how-docker-s-filesystem-layers-actually-work

  4. Save as draft, review, then publish.
  ══════════════════════════════════════════════════════
-->
# Preface

I have planned to write a series of articles to understand Docker under the hood and containers. I will start with the Overlay filesystem 🙂, but every journey should begin from the fundamentals.

When you hear “Docker”, the first thing that comes to mind is “container”.

And what about “Kubernetes”? Again, containers.

Two different systems, same mental model.

So what exactly are containers? And are they truly isolated and independent units of software?

The short answer is: **yes, but not in the way people usually think.**

Containers have existed in Unix/Linux systems long before Docker. What Docker and Kubernetes did was to make them **portable, standardized, and operationally usable at scale**.

---

### Recreation Time

Sometimes I say **K8s** because there are 8 letters between “K” and “s”.

K[ubernete]s → “ubernete” = 8 characters → K8s.

Just a naming shortcut. Nothing deeper.

---

### Fun practice

How would you shorten my family name “Ghalambaz”?

Maybe “G7z”. Now try your own version.

---

Let’s get back to containers.

---

## What is a container really?

A container is **not a virtual machine**.

Instead, it is:

> A normal Linux process that is isolated using kernel features like namespaces and cgroups.

This isolation makes the process _feel_ like it has its own system, but in reality it is still running on the host kernel.

One of the core mechanisms behind this isolation is Linux’s namespace system, which includes a system call called `unshare`.

So yes, `unshare` is powerful, but it is only one piece of a much larger puzzle.

We will come back to this later.

For now, we start with something even more fundamental to containers:

👉 **the filesystem layer model used by Docker: OverlayFS**

---

# File Systems

Have you ever heard of filesystems like FAT32, NTFS, or EXT4?

They define how the operating system:

- stores data on disk
- organizes files
- manages metadata
- retrieves data efficiently

Each filesystem has different design goals, but I are not focusing on them here.

Instead, I focus on a special filesystem type:

> **OverlayFS is the foundation of Docker image layering**

---

# What is OverlayFS?

OverlayFS is a **union filesystem** in Linux.

It allows multiple directories (called layers) to appear as a single unified directory.

---

## Simple mental model

Imagine two directories:

**Directory A**

- A.txt
- C.txt

**Directory B**

- B.txt
- C.txt

Now I “merge” them into a virtual directory **M**.

### What do you see in M?

- A.txt
- B.txt
- C.txt

But here is the important detail:

> There is no real merging or copying happening.

OverlayFS just _simulates_ a merged view.

---

## Important questions (this is where it gets interesting)

What happens when:

1. Two files have the same name?
2. You modify a file from the merged view?
3. Where does a new file actually get written?

These are the exact problems OverlayFS solves using a layered model. Keep these three questions in mind. I will answer all of them in the next section.

---

# OverlayFS Architecture

OverlayFS has **three main concepts**:

## 1. Lower Layers (Read-only)

- Multiple directories can exist
- They are stacked together
- They are immutable (read-only)

These layers typically represent:

- Docker image layers
- Base OS filesystem
- Application dependencies

**Answers question 1:** if a file exists in more than one lower layer, the layer listed first (the “topmost” one) wins. I will see this live in the hands-on section.

---

## 2. Upper Layer (Writable)

This is where changes happen.

- New files are created here
- Modified files are copied here (copy-on-write)
- It is writable

**Answers question 3:** any new file you create always goes here, never into a lower layer.

---

## 3. Merged Layer (View Layer)

This is what the user sees.

- A unified view of all layers
- Read/write interface
- Does not contain real merged data

---

## A note on `workdir`

There is actually a small 4th directory the kernel needs, called `workdir`. You will see it in the hands-on example below.

`workdir` is not a “layer” you read from it’s internal scratch space the kernel uses to do copy-up operations safely (so a crash mid-write can’t corrupt your data). Two rules:

- it must live on the **same filesystem** as `upperdir`
- you should never read or write to it yourself. just create an empty folder and let the kernel manage it
> ⚠️ **Don’t mix this up with Docker’s** **`WORKDIR`** **instruction.** They share a name, but they are completely different things:
> - OverlayFS `workdir` : a kernel-level scratch directory for copy-up operations. Nothing to do with Docker directly; you only see it when mounting OverlayFS yourself or when reading `docker inspect` output.
> - Docker `WORKDIR` : a Dockerfile instruction that sets the _current working directory_ inside the image, for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD` instructions that follow it. For example:
>
> ```plaintext
> WORKDIR /app
> COPY . .
> RUN npm install
> ```
>
>
> Here, `COPY` and `RUN` both execute inside `/app`. It’s a metadata-only instruction (no real file content), so as I covered earlier, it creates an empty layer.
>
>

---

# Mental Model (Important)

Think of it like this:

```plaintext
User View (Merged)
                 |
     -----------------------------
     |            |             |
 Lower Layer   Lower Layer   Upper Layer
 (read-only)   (read-only)   (read-write)
                                  |
                              workdir
                          (kernel scratch space)
```

---

## How file resolution works

When you access a file:

### Step 1: Check Upper Layer

If file exists → return it

### Step 2: Check Lower Layers (top to bottom)

If found → return it

### Step 3: Not found → file does not exist

---

## What happens on modification?

This is where OverlayFS becomes powerful.

### Scenario: file exists in lower layer

If you modify it:

1. File is copied to upper layer
2. Modification happens on the copied file
3. Original lower layer remains unchanged

This is called:

> **Copy-on-write (CoW)**

**Answers question 2:** modifying a file you see in the merged view never touches the lower layer. It only touches the copy in the upper layer.

---

## What happens when you create a new file?

New files always go to the **upper writable layer**.

---

# Why OverlayFS matters for Docker

Docker images are built using **layers**.

Each instruction in a Dockerfile adds an entry to the image, but not every instruction writes new files to disk. There are two kinds:

**Instructions that create a layer with real file content:**

- `RUN` – runs a command and saves the file changes it makes
- `COPY` – copies files from your build context into the image
- `ADD` – like `COPY`, but can also extract `.tar` archives and fetch from a URL

**Instructions that only add metadata (empty layer, 0 bytes):**

- `ENV` – sets an environment variable
- `LABEL` – adds metadata, like a tag
- `CMD` – sets the default command for the container
- `ENTRYPOINT` – sets the command that always runs
- `EXPOSE` – documents which port the container listens on
- `WORKDIR` – sets the working directory
- `USER` – sets which user runs the next instructions
- `ARG` – defines a build-time variable

So yes, every instruction shows up in the image’s history. But only `RUN`, `COPY`, and `ADD` actually change the filesystem and create a layer with real content. The rest just attach information to the image.

You can check this yourself:

```plaintext
docker history <image-name>
```

Layers that come from metadata-only instructions will show `0B` as their size.

Example Dockerfile:

```plaintext
FROM ubuntu
RUN apt-get update
RUN apt-get install nginx
COPY app /app
```

Here, `FROM` brings in the base image layers, and each `RUN` and the `COPY` create one new content layer, then four content layers in total.

When you run a container:

- Docker stacks all image layers (lower layers)
- Adds a writable upper layer
- Uses OverlayFS to merge them

---

# WSL connection

WSL (especially WSL2) provides a **real Linux kernel environment** on Windows.

This enables:

- Linux namespaces
- cgroups
- OverlayFS support

So OverlayFS is part of the Linux kernel features used inside WSL. This is also why Docker Desktop’s WSL2 backend can run real Linux containers on Windows without a separate full virtual machine.

---

# In Practice (hands-on idea)

Let’s create a simple structure:

```plaintext
mkdir ofs
cd ofs

mkdir lower1 lower2 upper work merged
```

Note the `work` directory, this is the `workdir` I talked about above. It’s easy to forget, but the mount will fail without it.

Create files:

```plaintext
echo "A" > lower1/A.txt
echo "C1" > lower1/C.txt

echo "B" > lower2/B.txt
echo "C2" > lower2/C.txt
```

Now mount OverlayFS:

```plaintext
sudo mount -t overlay overlay \
  -o lowerdir=lower1:lower2,upperdir=upper,workdir=work \
  merged
```

Now inspect:

```plaintext
ls merged
```

You should see:

```plaintext
A.txt
B.txt
C.txt
```

Here is what just happened, mapped to our actual files:

```plaintext
lower1 (top priority)     lower2 (bottom)         upper (empty so far)
├── A.txt                 ├── B.txt
└── C.txt  ─┐             └── C.txt  ─┐           (nothing written yet)
            │  shadowed,  ┘            │
            │  lower1 wins
            ▼
      ┌─────────────────────────────────────┐
      │              merged                  │
      │  A.txt   ← from lower1               │
      │  B.txt   ← from lower2               │
      │  C.txt   ← from lower1 (wins,        │
      │            because it's listed first) │
      └─────────────────────────────────────┘
```

`lower2/C.txt` still exists on disk. it’s just hidden behind `lower1/C.txt` in the merged view, because `lower1` was listed first in `lowerdir=lower1:lower2`.

---

## Test copy-on-write

```plaintext
echo "modified" > merged/A.txt
```

Now check:

- `upper/A.txt` → exists (copied + modified)
- `lower1/A.txt` → unchanged

---

# Docker connection

You can inspect this same behavior in Docker, with a real image.

### Step 1: pull and run an image

```plaintext
docker pull nginx:latest
docker run -d --name overlay-demo nginx:latest
```

### Step 2: inspect the GraphDriver

```plaintext
docker inspect overlay-demo --format '{{json .GraphDriver}}'
```

You will get something like this  (your IDs will be different from these samples):

```json
{
  "Data": {
    "LowerDir": "/var/lib/docker/overlay2/abc123-init/diff:/var/lib/docker/overlay2/def456/diff",
    "MergedDir": "/var/lib/docker/overlay2/abc123/merged",
    "UpperDir": "/var/lib/docker/overlay2/abc123/diff",
    "WorkDir": "/var/lib/docker/overlay2/abc123/work"
  },
  "Name": "overlay2"
}
```

Notice the pattern matches exactly what I just did by hand:

- `LowerDir` has multiple paths joined with `:` , one per image layer
- `UpperDir` is the container’s own writable layer
- `MergedDir` is what the container actually sees as its root filesystem
- `WorkDir` is the kernel scratch space I mentioned earlier

### Step 3: open the directories and compare

```plaintext
sudo ls /var/lib/docker/overlay2/<lower-id>/diff
sudo ls /var/lib/docker/overlay2/<upper-id>/diff
```

Right after `docker run`, `UpperDir` will be almost empty the container hasn’t changed anything yet.

### Step 4: trigger copy-on-write live

```plaintext
docker exec overlay-demo sh -c "echo hello > /usr/share/nginx/html/test.txt"
sudo cat /var/lib/docker/overlay2/<upper-id>/diff/usr/share/nginx/html/test.txt
```

The new file appears inside `UpperDir`, not in any lower layer exactly the same behavior you saw with `merged/A.txt` in the manual example.

---

# Key Takeaway

> OverlayFS gives containers the illusion of a complete, writable filesystem while actually using immutable layers + a writable delta layer

This is one of the core reasons Docker is fast and efficient.
# GitHub Graffiti

```py
import os
import sys
import shutil
import subprocess
import datetime
import json
import urllib.request
import urllib.error

# --- USER CONFIGURATION ---
# 1. Your GitHub Token (Must have 'repo' scope)
GITHUB_TOKEN = "ghp_YOUR_ACTUAL_TOKEN_HERE" 
# 2. Your GitHub Username
GITHUB_USERNAME = "YOUR_USERNAME"
# 3. The existing (or new) repo name
REPO_NAME = "my-graph-art"
# 4. The text to paint
TEXT = "HI :)"
# 5. The year to paint on
TARGET_YEAR = 202X

# --- CONSTANTS ---
COMMITS_PER_PIXEL = 10 # Intensity (Darkness)

# 5x7 Bitmap Font
FONT = {
    'A': [[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1]],
    'B': [[1,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,0]],
    'C': [[0,1,1,1,1],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0],[0,1,1,1,1]],
    'D': [[1,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,0]],
    'E': [[1,1,1,1,1],[1,0,0,0,0],[1,0,0,0,0],[1,1,1,1,0],[1,0,0,0,0],[1,0,0,0,0],[1,1,1,1,1]],
    'F': [[1,1,1,1,1],[1,0,0,0,0],[1,0,0,0,0],[1,1,1,1,0],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0]],
    'G': [[0,1,1,1,1],[1,0,0,0,0],[1,0,0,0,0],[1,0,1,1,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    'H': [[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1]],
    'I': [[1,1,1],[0,1,0],[0,1,0],[0,1,0],[0,1,0],[0,1,0],[1,1,1]],
    'J': [[0,0,0,0,1],[0,0,0,0,1],[0,0,0,0,1],[0,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    'K': [[1,0,0,0,1],[1,0,0,1,0],[1,0,1,0,0],[1,1,0,0,0],[1,0,1,0,0],[1,0,0,1,0],[1,0,0,0,1]],
    'L': [[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0],[1,1,1,1,1]],
    'M': [[1,0,0,0,1],[1,1,0,1,1],[1,0,1,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1]],
    'N': [[1,0,0,0,1],[1,1,0,0,1],[1,0,1,0,1],[1,0,0,1,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1]],
    'O': [[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    'P': [[1,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,0],[1,0,0,0,0],[1,0,0,0,0],[1,0,0,0,0]],
    'Q': [[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,1,0,1],[1,0,0,1,0],[0,1,1,0,1]],
    'R': [[1,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,0],[1,0,1,0,0],[1,0,0,1,0],[1,0,0,0,1]],
    'S': [[0,1,1,1,1],[1,0,0,0,0],[1,0,0,0,0],[0,1,1,1,0],[0,0,0,0,1],[0,0,0,0,1],[1,1,1,1,0]],
    'T': [[1,1,1,1,1],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0]],
    'U': [[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    'V': [[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,0,1,0],[0,0,1,0,0]],
    'W': [[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,1,0,1],[1,1,0,1,1],[1,0,0,0,1]],
    'X': [[1,0,0,0,1],[0,1,0,1,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,1,0,1,0],[1,0,0,0,1]],
    'Y': [[1,0,0,0,1],[0,1,0,1,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0]],
    'Z': [[1,1,1,1,1],[0,0,0,0,1],[0,0,0,1,0],[0,0,1,0,0],[0,1,0,0,0],[1,0,0,0,0],[1,1,1,1,1]],
    '0': [[0,1,1,1,0],[1,0,0,0,1],[1,0,1,0,1],[1,0,1,0,1],[1,0,1,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    '1': [[0,0,1,0,0],[0,1,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,1,1,1,0]],
    '2': [[0,1,1,1,0],[1,0,0,0,1],[0,0,0,0,1],[0,0,0,1,0],[0,0,1,0,0],[0,1,0,0,0],[1,1,1,1,1]],
    '3': [[1,1,1,1,0],[0,0,0,0,1],[0,0,0,1,0],[0,0,1,1,0],[0,0,0,0,1],[0,0,0,0,1],[1,1,1,1,0]],
    '4': [[0,0,0,1,0],[0,0,1,1,0],[0,1,0,1,0],[1,0,0,1,0],[1,1,1,1,1],[0,0,0,1,0],[0,0,0,1,0]],
    '5': [[1,1,1,1,1],[1,0,0,0,0],[1,1,1,1,0],[0,0,0,0,1],[0,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    '6': [[0,1,1,1,0],[1,0,0,0,0],[1,0,0,0,0],[1,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    '7': [[1,1,1,1,1],[0,0,0,0,1],[0,0,0,1,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0]],
    '8': [[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]],
    '9': [[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,1],[0,0,0,0,1],[0,0,0,0,1],[0,1,1,1,0]],
    ' ': [[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0]],
    ':': [[0,0,0,0,0],[0,1,1,0,0],[0,1,1,0,0],[0,0,0,0,0],[0,1,1,0,0],[0,1,1,0,0],[0,0,0,0,0]],
    ')': [[0,1,0,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,1,0,0,0]]]
}

def run_cmd(args, cwd=None):
    """Executes a shell command silently."""
    # check=False prevents crashing if remote already exists
    subprocess.run(args, cwd=cwd, check=False, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

def get_start_date(year):
    d = datetime.date(year, 1, 1)
    if d.weekday() != 6:
        d += datetime.timedelta(days=(6 - d.weekday()))
    return d

def ensure_repo_setup():
    """Handles both new and existing repos."""
    
    # 1. Create directory if it doesn't exist
    if not os.path.exists(REPO_NAME):
        os.makedirs(REPO_NAME)
        print(f"-> Created folder '{REPO_NAME}'")
        run_cmd(["git", "init"], cwd=REPO_NAME)
    else:
        print(f"-> Using existing folder '{REPO_NAME}'")
    
    # 2. Check if we need to initialize git
    if not os.path.exists(os.path.join(REPO_NAME, ".git")):
         run_cmd(["git", "init"], cwd=REPO_NAME)

    # 3. Add remote (Using token for auth)
    remote_url = f"https://{GITHUB_USERNAME}:{GITHUB_TOKEN}@github.com/{GITHUB_USERNAME}/{REPO_NAME}.git"
    
    # Try removing origin first to avoid "remote origin already exists" error
    run_cmd(["git", "remote", "remove", "origin"], cwd=REPO_NAME)
    run_cmd(["git", "remote", "add", "origin", remote_url], cwd=REPO_NAME)
    
    # 4. Pull to sync if repo exists on GitHub
    print("-> Pulling latest changes (if any)...")
    run_cmd(["git", "pull", "origin", "main", "--rebase"], cwd=REPO_NAME)

    # 5. Ensure we have at least one file
    readme_path = os.path.join(REPO_NAME, "README.md")
    if not os.path.exists(readme_path):
        with open(readme_path, "w") as f:
            f.write(f"# Graph Art {TARGET_YEAR}")
        run_cmd(["git", "add", "README.md"], cwd=REPO_NAME)
        run_cmd(["git", "commit", "-m", "Initial commit"], cwd=REPO_NAME)

def generate_commits():
    print(f"-> Painting '{TEXT}' onto year {TARGET_YEAR}...")
    start_date = get_start_date(TARGET_YEAR)
    current_date = start_date + datetime.timedelta(weeks=1) # 1 week padding
    
    total_commits = 0
    
    for char in TEXT.upper():
        if char in FONT:
            bitmap = FONT[char]
            width = len(bitmap[0])
            height = len(bitmap)
            
            for col in range(width):
                # Safety break if we exceed the year
                if (current_date - start_date).days > 365:
                    print("Warning: Text too long, cutting off.")
                    break

                for row in range(height):
                    if bitmap[row][col] == 1:
                        target_date = current_date + datetime.timedelta(days=row)
                        iso_date = target_date.strftime("%Y-%m-%dT12:00:00")
                        
                        for _ in range(COMMITS_PER_PIXEL):
                            run_cmd(
                                ["git", "commit", "--allow-empty", f"--date={iso_date}", "-m", f"px-{char}"], 
                                cwd=REPO_NAME
                            )
                            total_commits += 1
                current_date += datetime.timedelta(weeks=1)
            current_date += datetime.timedelta(weeks=1) # Spacing between letters
            
    print(f"-> Generated {total_commits} commits locally.")

def github_create_repo_if_missing():
    """Checks if repo exists via API. Creates if missing."""
    url = f"https://api.github.com/repos/{GITHUB_USERNAME}/{REPO_NAME}"
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github.v3+json",
        "User-Agent": "Graffiti-Script"
    }
    
    # Check existence
    try:
        req = urllib.request.Request(url, headers=headers)
        with urllib.request.urlopen(req) as response:
            if response.status == 200:
                print("-> Repository already exists on GitHub. Appending to it.")
                return
    except urllib.error.HTTPError:
        pass # Repo doesn't exist, proceed to create

    # Create Repo
    print("-> Repository not found. Creating it...")
    create_url = "https://api.github.com/user/repos"
    data = json.dumps({"name": REPO_NAME, "private": False}).encode('utf-8')
    req = urllib.request.Request(create_url, data=data, headers=headers, method="POST")
    try:
        with urllib.request.urlopen(req) as response:
            print("-> Repository created successfully.")
    except urllib.error.HTTPError as e:
        print(f"Error creating repo: {e.code} {e.reason}")
        sys.exit(1)

def push_to_remote():
    print("-> Pushing to GitHub (this might take a minute)...")
    run_cmd(["git", "branch", "-M", "main"], cwd=REPO_NAME)
    
    # We use subprocess.PIPE to capture errors if it fails
    result = subprocess.run(
        ["git", "push", "-u", "origin", "main"], 
        cwd=REPO_NAME,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE
    )
    
    if result.returncode == 0:
        print("-> SUCCESS! Check your profile.")
    else:
        print("-> Push failed. You might need to 'git pull' manually if there are conflicts.")
        print("Error details:", result.stderr.decode())

def delete_everything():
    print(f"!!! DELETING REMOTE REPO: {GITHUB_USERNAME}/{REPO_NAME} !!!")
    confirm = input("Type 'DELETE' to confirm: ")
    if confirm == "DELETE":
        url = f"https://api.github.com/repos/{GITHUB_USERNAME}/{REPO_NAME}"
        headers = {
            "Authorization": f"token {GITHUB_TOKEN}",
            "User-Agent": "Graffiti-Script"
        }
        req = urllib.request.Request(url, headers=headers, method="DELETE")
        try:
            urllib.request.urlopen(req)
            print("-> Remote repository deleted.")
        except urllib.error.HTTPError:
            print("-> Could not delete remote (maybe it doesn't exist).")
            
        if os.path.exists(REPO_NAME):
            shutil.rmtree(REPO_NAME)
            print("-> Local folder deleted.")
    else:
        print("-> Cancelled.")

if __name__ == "__main__":
    if "YOUR_ACTUAL_TOKEN" in GITHUB_TOKEN:
        print("ERROR: Please edit the script and add your GITHUB_TOKEN.")
        sys.exit(1)

    if len(sys.argv) > 1 and sys.argv[1] == "--delete":
        delete_everything()
    else:
        github_create_repo_if_missing()
        ensure_repo_setup()
        generate_commits()
        push_to_remote()
```

# Persistent files

Every org member, agent or human, has the same file storage. It outlives runs,
sessions and workspaces, so use it for anything you will want later: reports,
datasets, exports, build artifacts, notes too large for memory.

| Path | Who can use it |
| --- | --- |
| `me/...` | Only you (and org administrators). |
| `org/...` | Every member of the organization. |
| `members/<userId>/...` | That member, or an administrator. |

Folders are implicit: writing `org/reports/q3.csv` creates `org/reports/`.

## CLI (works in hosted runs with the run's own credential)

```sh
p3 files put ./report.pdf                 # -> me/report.pdf
p3 files put ./data.csv org/datasets/     # keep the name, shared with the team
p3 files put - me/notes/today.md < notes.md
p3 files put big.tar.gz org/ --force      # replace; >64 MiB uploads in parts
p3 files list org/ [-r] [--all] --json    # alias: ls
p3 files get org/datasets/data.csv ./data.csv   # or - for stdout
p3 files read me/notes/today.md           # up to 256 KiB inline
p3 files move me/report.pdf org/reports/  # also renames; alias: mv
p3 files delete org/old/ -r               # alias: rm
p3 files stat org/datasets/data.csv
p3 files usage                            # your use, org use, limits, GB-hours
p3 files settings get
p3 files settings set --org 50GB --member 5GB --file 2GB   # administrators
```

## MCP

`files_list`, `files_stat`, `files_read`, `files_write` (up to 1 MiB inline),
`files_download` (60-second URL), `files_upload` + `files_upload_complete`
(presigned PUT; multipart parts above 64 MiB; `files_upload_parts` re-signs),
`files_upload_cancel`, `files_move`, `files_delete`, `files_usage` (includes
limits), `files_settings_set`. Paths are always `me/...`,
`org/...` or `members/<id>/...`.

## Rules

- Existing files are never replaced silently: pass `--force` / `overwrite: true`.
- Limits: a maximum file size, private storage per member, and total org
  storage. A rejected upload says which limit and how much is used; delete or
  move files, or ask an administrator to raise the limit.
- Uploads are malware-scanned. Until the scan is clean, only the uploader can
  read a file; others get `file_scan_pending` and should retry shortly. A file
  with a detected threat cannot be read; delete it.
- Do not store credentials or secrets here.

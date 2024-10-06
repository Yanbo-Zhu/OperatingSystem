
使用 getopts  

Das POSIX-konforme Shell-Builtin [getopts](https://www.gnu.org/software/bash/manual/bash.html#index-getopts) kann _short options_ der Form `-a` (ohne Argument) oder `-b 123` (mit erforderlichem Argument) auswerten.



例子: 

timestamp.sh hängt an die Namen aller regulären Dateien im aktuellen Verzeichnis deren Änderungsdatum im Format YYYYMMDD an, separiert mit _. Wenn im Dateinamen schon ein Datums vorhanden ist, wird dieses aktualisiert. Das Script hat mehrere Optionen:

    -t (time) hängt auch die Zeit mit an: YYYYMMDD-HH:MM.

    -p (prefix) wird das Datum dem Dateinamen vorangestellt, nicht angehängt

    -r (remove) entfernt einen ggf. vorhandener Datums-Suffix/-Präfix; kein Fehler, wenn es keinen gibt.

    -h (help) gibt einen kurzen Hilfetext aus.

Die Optionen -r und -t sind jeweils mit -p kombinierbar. Unzulässige oder unlogische Kombinationen führen zu einer Fehlermeldung. Die Kommandos grep, sed, awk dürfen Sie nicht verwenden. Nutzen Sie

    das Bash-Builtin getopts zur Analyse der gegebenen Optionen

    regular expression matching und die Variable BASH_REMATCH(siehe Bash Reference Manual), um im Dateinamen einen schon vorhandenen Datums-Suffix/-Präfix zu suchen und zu extrahieren.

    date -r um den Zeitstempel der letzten Änderung in das gewünschte Format umzuwandeln

---
To implement the `timestamp.sh` script with the specified functionality, we can use Bash's `getopts` to parse the options, regular expression matching to identify any existing date suffixes/prefixes, and the `date` command to format the modification date of files. Here's how this can be done step-by-step:

---


```sh

#!/bin/bash

# Function to display help text
show_help() {
    echo "Usage: $0 [OPTIONS]"
    echo "Options:"
    echo "  -t        Append time to the timestamp (format: YYYYMMDD-HH:MM)"
    echo "  -p        Use the timestamp as a prefix instead of a suffix"
    echo "  -r        Remove existing date from the filename (if any)"
    echo "  -h        Show this help message"
}

# Initialize variables for options
time_flag=0
prefix_flag=0
remove_flag=0

# Parse options using getopts
while getopts "tprh" opt; do
    case $opt in
        t) time_flag=1 ;;
        p) prefix_flag=1 ;;
        r) remove_flag=1 ;;
        h) show_help; exit 0 ;;
        *) echo "Invalid option"; show_help; exit 1 ;;
    esac
done

# Function to extract and remove date from the filename (if it exists)
remove_date() {
    local filename="$1"
    local regex="(.*)(_[0-9]{8}(-[0-9]{2}:[0-9]{2})?|[0-9]{8}(-[0-9]{2}:[0-9]{2})?_)(.*)"
    if [[ $filename =~ $regex ]]; then
        echo "${BASH_REMATCH[1]}${BASH_REMATCH[4]}"
    else
        echo "$filename"
    fi
}

# Function to append or prepend timestamp to the filename
add_timestamp() {
    local filename="$1"
    local timestamp=$(date -r "$filename" +"%Y%m%d")

    # Add time if the -t option was used
    if [ $time_flag -eq 1 ]; then
        timestamp=$(date -r "$filename" +"%Y%m%d-%H:%M")
    fi

    # Modify filename based on -p (prefix) or default (suffix)
    if [ $prefix_flag -eq 1 ]; then
        echo "$timestamp"_"$filename"
    else
        # Extract file extension if it exists
        local base="${filename%.*}"
        local ext="${filename##*.}"
        if [ "$base" == "$ext" ]; then
            # No extension case
            echo "${filename}_$timestamp"
        else
            # With extension
            echo "${base}_$timestamp.$ext"
        fi
    fi
}

# Process all regular files in the current directory
for file in *; do
    if [ -f "$file" ]; then
        original_filename="$file"
        
        # If -r is set, remove the existing date
        if [ $remove_flag -eq 1 ]; then
            original_filename=$(remove_date "$file")
        fi

        # Add timestamp (or update the date) to the file
        new_filename=$(add_timestamp "$original_filename")

        # Rename the file
        mv "$file" "$new_filename"
    fi
done
```


Explanation:

    Options Parsing:
        The script uses getopts to handle the -t, -p, -r, and -h options.
        These flags are stored in variables (time_flag, prefix_flag, remove_flag) to be used later.

    Remove Date (remove_date function):
        Uses a regular expression to detect existing date patterns (either suffix or prefix) in the filename and strips them out.

    Add Timestamp (add_timestamp function):
        The date -r "$filename" command is used to get the modification date of the file in the required format (either YYYYMMDD or YYYYMMDD-HH:MM depending on the -t option).
        The new filename is created by appending (or prepending) the timestamp.

    File Renaming:
        The script iterates over all regular files (-f "$file") in the current directory.
        It removes the date if -r is specified and then either appends or prepends the timestamp, renaming the file accordingly using mv.

Options:

    -t: Adds time (HH:MM) to the date.
    -p: Uses the timestamp as a prefix instead of a suffix.
    -r: Removes an existing date from the filename.
    -h: Displays the help message.
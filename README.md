# PS5-BAR-Reader
(DOESN'T WORK NOW, IN PROGRESS)
This repository contains code to help you verify and restore your PS5 backup data, particularly if you encounter the CE-100511-8 error during the restore process. The error (in my case) always occurs at constant % progress during restoration, indicating a possible issue with the transition between archive files.

## Background

After successfully backing up my PlayStation 5, I encountered the CE-100511-8 error while trying to restore the backup. Despite researching online and checking PlayStation's official resources, I couldn't find a solution.

### Troubleshooting Steps

I attempted several troubleshooting steps, including changing the USB format, but the issue persisted:
- **FAT32**: Error persists
- **ExFAT**: Error persists
- **NTFS**: PS5 doesn't recognize this format

The problem consistently occurred at 21% of the restoration process. After calculating, I realized that this coincides with the transition from `archive.dat` to `archive0001.dat`, which led me to investigate further.

## Project Overview

This project includes code to:
1. **Read and Parse PS5 Backup Files**: The code reads `archive.dat` files using the structure provided by PSDevWiki.

## Structures

### Archive Header Structure

The archive files follow a specific format, as outlined by the PSDevWiki. The code reads and parses the following structures:

```cpp
#define ARCHIVE_HEADER_MAGIC             0x0000464143454953
#define ARCHIVE_HEADER_SIZE              0x58
#define ARCHIVE_SEGMENT_TABLE_SIZE       0x40
#define ARCHIVE_SEGMENT_SIGNATURE_SIZE   0x30
#define ARCHIVE_MAGIC_SIZE               0x08

typedef union caf_header_s {
    struct {
        uint8_t magic[ARCHIVE_MAGIC_SIZE]; // SIECAF
        uint64_t version; // 1
        uint64_t unk1; // 1
        uint64_t unk2; // 1
        uint8_t unk3[0x10];
        uint32_t unk4; // 0x75BF88
        uint32_t unk5; // 0x75BF89
        uint32_t unk6; // 0x7FBF8A
        uint32_t padding;
        uint64_t num_segments;
        uint64_t file_offset;
        uint64_t file_size;
    };
    uint8_t raw[ARCHIVE_HEADER_SIZE];
} caf_header_t;

typedef union caf_segment_table_s {
    struct {
        uint64_t index;
        uint64_t data_offset;
        uint64_t aligned_data_size;
        uint64_t signature_key_id; // (0x0000000000000003)
        uint64_t enc_key_id; // (0x0000000000000001)
        uint8_t enc_iv[0x10]; // (null) Used to be filled on PS4, maybe replaced from header?
        uint64_t unaligned_data_size;
    };
    uint8_t raw[ARCHIVE_SEGMENT_TABLE_SIZE];
} caf_segment_table_t;

typedef union caf_segment_signature_s {
    struct {
        uint64_t index;
        uint8_t signature[0x10];
        uint8_t padding[0x18];
    };
    uint8_t raw[ARCHIVE_SEGMENT_SIGNATURE_SIZE];
} caf_segment_signature_t;
```

## Usage

To use the code in this repository:

1. **Clone the Repository**:
    ```bash
    git clone https://github.com/yourusername/PS5-BAR-Reader.git
    cd PS5-BAR-Reader
    ```

2. **Compile the Code**:
    Ensure you have a C++ compiler and OpenSSL installed. Then compile the code:
    ```bash
    g++ -o ps5_bar_reader main.cpp -lssl -lcrypto
    ```

3. **Run the Program**:
    ```bash
    ./ps5_bar_reader 
    ```

    The program will:
    - Read the headers of the all archive.dat files.

## Contributing

Feel free to contribute to this project by submitting issues or pull requests. Any improvements or additional features are welcome.

## References

- [PSDevWiki - Archive.dat](https://www.psdevwiki.com/ps5/Archive.dat)

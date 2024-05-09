Uses SD card(also nand ssd) instead of ssd as persistent storage

Introduce this design from bottom up:
## Earth Layer
### earth/sd/ directory includes sd_init.c sd_rw.c sd_utils.c
sd_utils.c is a wrapper about executing commands by sending/receving data bytes to registers (MMIO, so no need to use separate port-related instructions) 
sd_init.c and sd_rw.c uses sd_exec_cmd(...) and sd_exec_acmd(...) from sd_utils.c, for SD card initialization(for example, set blk size to 512bytes) and executing block read/write commands. 
Note that currently, sd_rw.c is very naive in that it writes/reads blocks one at a time. (TODO: better way is using hardware provided abilities to read/write multiple blocks at a time)
### Core of filesystem:
file mapping: file + offset => block no
Different data structures are tradeoff between different workloads:
Extent tree, Radix tree, Cuckoo hash table
Per-file or Global
### earth/dev_disk.c
more general interface of disk read/write.
different hardware(FLASH_ROM, SD_CARD) has different implementation of interface:
earth->disk_read(...), earth->disk_write(...)
## Grass Layer
### library/file directory
#### disk.c
Encapsulates earth->disk_read(...), earth->disk_write(...)
#### inode.h
```c
#define NINODES  128

typedef struct inode_store {
    int (*getsize)(struct inode_store *this_bs, unsigned int ino);
    int (*setsize)(struct inode_store *this_bs, unsigned int ino, block_no newsize);
    int (*read)(struct inode_store *this_bs, unsigned int ino, block_no offset, block_t *block);
    int (*write)(struct inode_store *this_bs, unsigned int ino, block_no offset, block_t *block);
    void *state;
} inode_store_t;

typedef inode_store_t *inode_intf;    /* inode store interface */

inode_intf fs_disk_init();
...
```
#### fs.h
defines below data structure:
super block(block #0), fs_inode(block #2 to #11), fs_dirent(32 bytes each entry), in-memory fs(one buffer used to cache inode block, one bitmap block cache(block #1))
```c
/* Superblock */
  struct fs_super {
      m_uint32 magic;       /*==0x6640*/
      m_uint32 disk_size;   /* in blocks */
  
      /* pad out to an entire block */
      char pad[BLOCK_SIZE - 2 * sizeof(m_uint32)];
  };
  
  struct fs_inode {
      m_uint32 mode;
      m_uint32 size;   /* file size in bytes */
      m_uint32 pads[3];
      m_uint32 ptrs[NUM_PTRS]; /* direct data block pointer */
      m_uint32 indirect_ptr; /* indirect pointer */
  };
  
  /* Entry in a directory */
  struct fs_dirent {
      m_uint32 valid : 1;
      m_uint32 inum  : 31; 
      char name[28];         /* with trailing NUL */
  };
  
  /* in-memory fs data structure*/
  struct fs_struct {
      int super_blk_id;
      int total_blks;
      int avail_blks;
      // page caches
      unsigned char bitmap[BLOCK_SIZE];
      int buffer_blk_id;
      unsigned char buffer_cache[BLOCK_SIZE];
  };
```

#### fs.c
Util methods of implementing filesystem:

block_read(...), block_write(...) just calls earth->disk_read(...), earth->disk_write(...)

alloc_block(...), free_block(...) are used for allocating, freeing blocks by checking bitmap block(which is cached in global variable of type fs_struct when initializing through fs_init())

lookup_name(...) used for searching files under dir

fs_dir_lookup(...)  gets a dir_ino and path as parameters, returning the inode number of the found file. This method calls lookup_name method

#### fs_rw.c
Util methods for read and write

fs_init()
fs_read() and fs_write(): inode no + offset + len + buf as parameters
fs_write(), same as above




### apps/system/
Uses methods from library/file directory to handle different kinds of requests. For example, SYS_FILE process calls fs_read() method when reading from file with inode number that >= FS_ROOT_INODE, otherwise uses interfaces implemented for treedisk_fs (Another fs), see below 2 files
#### sys_file.c
#### sys_dir.c
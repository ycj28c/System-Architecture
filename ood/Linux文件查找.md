## ood 设计一个Linux find

### 要求
implemnet linux find command as an api ,the api willl support finding files that has given size requirements and a file with a certain format like

1.find all file >5mb  
2.find all xml
Assume file class
{
get name()
directorylistfile()
getFile()
create a library flexible that is flexible
Design clases,interfaces.

### 考点
1. Obviously coming straight to the right design (encapsulating the Filtering logic into its own interface etc...), with an explanation on why this approach is good. I'm obviously open to alternate approaches as long as they are as flexible and elegant.  
创建一个Filter 的base class,  创造子类来表示不同的filter 条件( size, type)  
2. Implement boolean logic: AND/OR/NOT, here I want to see if the candidate understands object composition  
用Decorator Pattern 实现Predicate (逻辑 与或非)  
3. Support for symlinks. Rather than seeing the implementation (which I don't really care about) I want to understand the tradeoffs of adding yet another parameter to the find method VS other options (eg. Constructor). Keep adding params to a method is usually bad.  
4. How to handle the case where the result is really big (millions of files), and you may not be able to put all of them into a List. 
 
### 代码示例 
```
class File {
    String name;
    int size;
    int type;
    boolean isDirectory;
    File[] children;
}

abstract class Filter {
    abstract boolean apply(File file);
}

class MinSizeFilter extends Filter {

    int minSize;

    public MinSizeFilter(int minSize) {
        this.minSize = minSize;
    }

    @Override
    boolean apply(File file) {
        return file.size > minSize;
    }
}

class TypeFilter extends Filter {

    int type;

    public TypeFilter(int type) {
        this.type = type;
    }

    @Override
    boolean apply(File file) {
        return file.type == type;
    }
}

class FindCommand {

    public List<File> findWithFilters(File directory, List<Filter> filters) {
        if (!directory.isDirectory) {
            return new NotADirectoryException();
        }
        List<File> output = new ArrayList<>();
        findWithFilters(directory, filters, output);
        return output;
    }

    private void findWithFilters(File directory, List<Filter> filters, List<File> output) {
        if (directory.children == null) {
            return;
        }
        for (File file : directory.children) {
            if (file.isDirectory) {
                findWithFilters(file, filters, output);
            } else {
                boolean selectFile = true;
                for (Filter filter : filters) {
                    if (!filter.apply(file)) {
                        selectFile = false;
                    }
                }
                if (selectFile) {
                    output.add(file);
                }
            }
        }
    }
}
```

### reference
[请教一道亚麻最近高频的OOD linux find题目](https://www.1point3acres.com/bbs/thread-431401-1-1.html)  
[Search directories recursively for file in Java](https://mkyong.com/java/search-directories-recursively-for-file-in-java/)
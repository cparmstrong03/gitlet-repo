package gitlet;

import java.io.File;
import java.io.IOException;
import java.io.Serializable;

public class Branch implements Serializable {

    /**this is.*/
    static final File REPO_FOLDER = new File(".", ".gitlet");
    /**this is.*/
    static final File BRANCH_FOLDER = new File(REPO_FOLDER, "branches");
    /**this is.*/
    static final File ACTIVE_FOLDER = new File(BRANCH_FOLDER, "Active");
    /**this is.*/
    static final File ACTIVE = new File(ACTIVE_FOLDER, "active");

    public Branch(File pointer, String name) {
        _Pointer = pointer;
        _Name = name;
    }


    public void writeBranch() throws IOException {
        File thisBranch = new File(BRANCH_FOLDER, getName());
        if (!thisBranch.exists()) {
            thisBranch.createNewFile();
        }
        Utils.writeObject(thisBranch, this);

    }

    public static Branch readBranch(String name) {
        File f = new File(BRANCH_FOLDER, name);
        if (!f.exists()) {
            throw new GitletException("This branch doesnt' exist " + name);
        }
        Branch b = Utils.readObject(f, Branch.class);
        return b;
    }




    public void setPointer(File c) {
        _Pointer = c;
    }

    public File getPointer() {
        return _Pointer;
    }

    public String getName() {
        return _Name;
    }

    /**this is.*/
    private String _Name;
    /**this is.*/
    private File _Pointer;

}

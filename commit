package gitlet;

import java.io.File;
import java.io.IOException;
import java.io.Serializable;
import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;
import java.security.NoSuchAlgorithmException;
import java.util.Date;

import java.text.SimpleDateFormat;

import java.util.Map;
import java.util.TreeMap;

public class Commit implements Serializable {
    /**this is.*/
    static final File REPO_FOLDER = new File(".", ".gitlet");
    /**this is.*/
    static final File COMMIT_FOLDER = new File(REPO_FOLDER, "Commits");




    public Commit() {
        _Message = "initial commit";
        _Date = new Date(0, 0, 0, 0, 0, 0);
        _Parent = null;
        _ParentSecond = null;
        String hashMessage = _Message + _Date.toString();
        _Hash = toSHA1(hashMessage.getBytes(StandardCharsets.UTF_8));
        _Map = new TreeMap<>();
        _Map.put(_Hash, 0);

    }

    public Commit(File parent, Date d, String message)
            throws NoSuchAlgorithmException, UnsupportedEncodingException {
        _Date = d;
        _Parent = parent;
        _ParentSecond = null;
        _Message = message;
        String hashMessage = _Message + _Date.toString();
        if (_Parent != null) {
            hashMessage += _Parent.getName();
        }

        _Hash = toSHA1(hashMessage.getBytes(StandardCharsets.UTF_8));
        _Map = new TreeMap<>();

        TreeMap<String, Integer> oldMap =
                new TreeMap<>(Commit.readCommit(parent.getName()).getMap());
        for (Map.Entry<String, Integer> entry : oldMap.entrySet()) {
            _Map.put(entry.getKey(), entry.getValue() + 1);
        }
        _Map.put(_Hash, 0);

    }

    public Commit(File parent, File parent2, Date d, String message)
            throws NoSuchAlgorithmException, UnsupportedEncodingException {
        _Date = d;
        _Parent = parent;
        _ParentSecond = parent2;
        _Message = message;
        String hashMessage = _Message + _Date.toString();
        if (_Parent != null) {
            hashMessage += _Parent.getName();
        }
        _Hash = toSHA1(hashMessage.getBytes(StandardCharsets.UTF_8));
        _Map = new TreeMap<>();

        TreeMap<String, Integer> oldMap =
                new TreeMap<>(Commit.readCommit(parent.getName()).getMap());
        for (Map.Entry<String, Integer> entry : oldMap.entrySet()) {
            _Map.put(entry.getKey(), entry.getValue() + 1);
        }

        TreeMap<String, Integer> secondOldMap =
                new TreeMap<>(Commit.readCommit(parent2.getName()).getMap());
        for (Map.Entry<String, Integer> entry : secondOldMap.entrySet()) {
            _Map.put(entry.getKey(), entry.getValue() + 1);
        }
        _Map.put(_Hash, 0);

    }

    public void makeFolderWithMessage(String s) {
        File thisCommit = new File(COMMIT_FOLDER, s);

        if (!thisCommit.exists()) {
            thisCommit.mkdir();

        } else {
            throw new RuntimeException(
                    "why are we trying to make"
                            + " this commit  into a folder twice: "
                            + s);
        }
    }

    public void writeCommit() throws IOException {
        File tHISCOMMIT = new File(COMMIT_FOLDER, getHash());
        File thisCommit = new File(tHISCOMMIT, "thisCommitObject");
        if (!thisCommit.exists()) {
            thisCommit.createNewFile();
        }
        Utils.writeObject(thisCommit, this);
    }

    public static Commit readCommit(String name) {
        File file = new File(COMMIT_FOLDER, name);
        File f = new File(file, "thisCommitObject");
        if (!f.exists()) {
            throw new GitletException("This commit doesnt' exist " + name);
        }

        Commit b = Utils.readObject(f, Commit.class);
        return b;
    }

    public static String toSHA1(byte[] convertme) {
        return Utils.sha1(convertme);

    }




    public String getTimeFormatted() {
        Date d = getDate();
        SimpleDateFormat sdf = new
                SimpleDateFormat(
                        "EEE MMM dd HH:mm:ss yyyy Z");
        return sdf.format(d);
    }

    public Date getDate() {
        return _Date;
    }

    public String getHash() {
        return _Hash;
    }

    public String getMessage() {
        return _Message;
    }

    public File getParent() {
        return _Parent;
    }

    public File getSecondParent() {
        return _ParentSecond;
    }
    public TreeMap<String, Integer> getMap() {
        return _Map;
    }


    /**this is.*/
    private TreeMap<String, Integer> _Map;
    /**this is.*/
    private File _Parent;
    /**this is.*/
    private File _ParentSecond;
    /**this is.*/
    private Date _Date;
    /**this is.*/
    private String _Hash;
    /**this is.*/
    private String _Message;


}

package gitlet;

import java.io.File;
import java.util.Map;
import java.util.TreeMap;
import java.io.IOException;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Date;
import java.util.Set;



/** Driver class for Gitlet, the tiny stupid version-control system.
 *  @author cparmstrong
 */
public class Main {


    /** Main metadata folder. */
    static final File CWD = new File(".");

    /** Main metadata folder. */
    static final File REPO_FOLDER = new File(".gitlet");
    /** Main metadata folder. */
    static final File STAGING_FOLDER = new File(REPO_FOLDER, "Staging Area");
    /** Main metadata folder. */
    static final File REMOVE_FOLDER = new File(REPO_FOLDER, "Remove Stage");
    /** Main metadata folder. */
    static final File COMMIT_FOLDER = new File(REPO_FOLDER, "Commits");
    /** Main metadata folder. */
    static final File BRANCH_FOLDER = new File(REPO_FOLDER, "branches");
    /** Main metadata folder. */
    static final File ACTIVE_FOLDER = new File(BRANCH_FOLDER, "Active");
    /** Main metadata folder. */
    static final File ACTIVE = new File(ACTIVE_FOLDER, "active.txt");



    /** Usage: java gitlet.Main ARGS, where ARGS contains
     *  <COMMAND> <OPERAND> .... */
    public static void main(String... args)
            throws IOException, NoSuchAlgorithmException {

        if (args.length == 0) {
            System.out.println("Please enter a command.");
            return;
        } else if (!args[0].equals("init") && !REPO_FOLDER.exists()) {
            System.out.println("Not in an initialized Gitlet directory.");
            return;
        }
        if (fakeMain(args)) {
            return;
        }

        switch (args[0]) {

        case "test":
            Branch b = Branch.readBranch(args[1]);
            thisIsAtestingMethod(b);
            break;
        case "reset":
            reseto(args);
            break;
        case "merge":
            if (listOfFiles(STAGING_FOLDER).length != 0
                    || listOfFiles(REMOVE_FOLDER).length != 0) {
                System.out.println("You have uncommitted changes.");
                return;
            }
            File doesBranchExist = new File(BRANCH_FOLDER, args[1]);
            if (!doesBranchExist.exists()) {
                System.out.println("A branch with that name does not exist.");
                return;

            }
            if (checkinStuff1(args)) {
                return;
            }
            if (checkinStuff2(args)) {
                return;
            }
            Branch mergeBranch = Branch.readBranch(args[1]);
            merge(mergeBranch);
            break;
        default:
            System.out.println("No command with that name exists.");
            break;

        }
    }


    public static void setupPersistence() throws IOException {

        if (!REPO_FOLDER.exists()) {

            REPO_FOLDER.mkdir();
        }

        if (!BRANCH_FOLDER.exists()) {
            BRANCH_FOLDER.mkdir();
            ACTIVE_FOLDER.mkdir();
            ACTIVE.createNewFile();
        }


        if (!STAGING_FOLDER.exists()) {
            STAGING_FOLDER.mkdir();
        }
        if (!REMOVE_FOLDER.exists()) {
            REMOVE_FOLDER.mkdir();
        }
        if (!COMMIT_FOLDER.exists()) {
            COMMIT_FOLDER.mkdir();
        }
    }

    /* a call to this should
     * --create a new repository
     * --make and save an empty commit "initial commit"
     * --make a branch called master*/
    public static void initialize()
            throws IOException, NoSuchAlgorithmException {

        File f = commit("initial commit");
        Branch b = new Branch(f, "master");
        b.writeBranch();
        setActive("master");

    }

    public static void add(String[] args) throws IOException {

        File inRemoveArea = getFileFromFolder(REMOVE_FOLDER, args[1]);
        if (inRemoveArea.exists()) {
            inRemoveArea.delete();
        }
        File f = getFileFromFolder(CWD, args[1]);
        File prev =
                getFileFromFolder
                        (getActiveCommit(getActiveBranch()), args[1]);
        if (prev.exists()) {
            if (compare(f, prev)) {
                File inStagingArea = getFileFromFolder(STAGING_FOLDER, args[1]);
                if (inStagingArea.exists()) {
                    inStagingArea.delete();
                }
                return;
            }
        }
        if (!f.exists()) {
            System.out.println("File does not exist.");
            return;
        }


        copyTo(CWD, STAGING_FOLDER, args[1]);
    }

    public static void remove(String fileName) throws IOException {
        File fileInDir = getFileFromFolder(CWD, fileName);
        File fileInStage = getFileFromFolder(STAGING_FOLDER, fileName);
        File recentCommit = (getActiveCommit(getActiveBranch()));
        File fileInPrev = getFileFromFolder(recentCommit, fileName);


        if (!(fileInStage.exists() || fileInPrev.exists())) {
            System.out.println("No reason to remove the file.");
            return;
        }
        if (fileInStage.exists()) {
            fileInStage.delete();
        }
        if (fileInPrev.exists()) {
            copyTo(recentCommit, REMOVE_FOLDER, fileName);
            if (fileInDir.exists()) {
                fileInDir.delete();
            }
        }

    }


    public static void
        copyTo(File location, File destination, String fileName)
            throws IOException {
        File f = new File(location, fileName);

        if (!f.exists()) {
            throw new GitletException("there should be a fi"
                    +
                    "le of this name and there isnt: " + fileName);
        }

        File fPlaced = new File(destination, fileName);
        if (!fPlaced.exists()) {
            fPlaced.createNewFile();
        } else {

            doNothing();
        }

        Utils.writeContents(fPlaced, Utils.readContentsAsString(f));
    }

    public static File getFileFromFolder(File folder, String nameOfFile) {
        File f = new File(folder, nameOfFile);
        return f;
    }

    public static File commit(String commitMessage)
            throws IOException, NoSuchAlgorithmException {


        File f;
        Date d = new Date();
        Commit c = null;
        if (commitMessage.equals("initial commit")) {
            c = new Commit();
            f = makeFolderWithMessage(c.getHash());
            c.writeCommit();
        } else {

            c = new Commit(getActiveCommit
                    (getActiveBranch()), d, commitMessage);
            f = makeFolderWithMessage(c.getHash());
            Branch b = getActiveBranch();
            File activeCom = getActiveCommit(b);


            File[] old = activeCom.listFiles();

            for (int i = 0; i < old.length; i++) {
                File curr = old[i];
                if (!curr.getName().equals("thisCommitObject")) {
                    File inRemoveFolder =
                            new File(REMOVE_FOLDER, curr.getName());
                    if (!inRemoveFolder.exists()) {
                        copyTo(activeCom, f, curr.getName());
                    }

                }

            }

            b.setPointer(f);
            b.writeBranch();
            c.writeCommit();

        }


        File[] files = listOfFiles(STAGING_FOLDER);

        for (int i = 0; i < files.length; i++) {
            File curr = files[i];

            copyTo(STAGING_FOLDER, f, curr.getName());
        }


        return f;
    }

    public static void saveThisCommitObject(File commitFolder) {

    }

    public static File makeFolderWithMessage(String s) {
        File thisCommit = new File(COMMIT_FOLDER, s);
        if (!thisCommit.exists()) {
            thisCommit.mkdir();
        } else {
            throw new RuntimeException("why are we trying to ma"
                    +
                    "ke this commit  into a folder twice: " + s);
        }
        return thisCommit;
    }

    public static File[] listOfFiles(File folder) {
        File[] list = folder.listFiles();
        return list;
    }

    public static void clearStagingArea() {
        File[] list = listOfFiles(STAGING_FOLDER);
        for (File f : list) {
            f.delete();
        }
    }

    public static void clearRemoveArea() {
        File[] list = listOfFiles(REMOVE_FOLDER);
        for (File f : list) {
            f.delete();
        }
    }

    public static Branch getActiveBranch() {
        File branchWeWant =
                new File(BRANCH_FOLDER, Utils.readContentsAsString(ACTIVE));
        Branch active = Utils.readObject(branchWeWant, Branch.class);
        return active;
    }
    public static File getActiveCommit(Branch active) {

        File f = active.getPointer();
        return f;
    }

    public static void setActive(String s) {
        Utils.writeContents(ACTIVE, s);
    }

    public static void checkout(String s, File commit) throws IOException {
        File location = commit;
        File lookingFor = getFileFromFolder(location, s);
        if (lookingFor.exists()) {
            copyTo(location, CWD, s);
        } else {
            System.out.println("File does not exist in that commit.");
        }

    }

    public static void checkout(String s, String commitHash)
            throws IOException {

        File thisCom = new File(COMMIT_FOLDER, commitHash);
        if (!thisCom.exists()) {
            System.out.println("No commit with that id exists.");
            return;
        }
        checkout(s, thisCom);


    }

    public static void reset(String commitHash) throws IOException {
        Branch b = getActiveBranch();
        File newHead = new File(COMMIT_FOLDER, commitHash);
        File oldHead = getActiveCommit(b);

        File[] makeSureTracked = newHead.listFiles();
        for (File newFile : makeSureTracked) {
            File problemIfNotExist = new File(oldHead, newFile.getName());
            File cwdFile = new File(CWD, newFile.getName());
            if (cwdFile.exists()) {
                if (!compare(cwdFile, newFile)) {
                    if (!problemIfNotExist.exists()) {
                        System.out.println("There is an"
                                +
                                " untracked file in the way;"
                                + " delete it, or add and commit it first.");
                        return;
                    } else if (!compare(problemIfNotExist, cwdFile)) {
                        System.out.println("There "
                                +
                                "is an untracked file in the way;"
                                        +
                                        " delete it, or add"
                                        +
                                        " and commit it first.");
                        return;
                    }
                }
            }
        }


        b.setPointer(newHead);
        checkoutBranch(b.getName());
        b.writeBranch();

    }

    public static void checkoutBranch(String branchName) throws IOException {

        File newBranchFile = new File(BRANCH_FOLDER, branchName);


        Branch b = Utils.readObject(newBranchFile, Branch.class);
        File newCommit = getActiveCommit(b);
        File prev = getActiveCommit(getActiveBranch());
        File[] prevFiles = listOfFiles(prev);
        File[] newFiles = listOfFiles(newCommit);

        for (int i = 0; i < newFiles.length; i++) {
            File first = newFiles[i];
            String s1 = first.getName();
            boolean found = false;
            for (int j = 0; j < prevFiles.length; j++) {
                File second = prevFiles[j];
                String s2 = second.getName();
                if (s1.equals(s2)) {
                    found = true;
                }
            }
            if (!found) {
                File couldBeErrorCase = new File(CWD, s1);
                if (couldBeErrorCase.exists()) {
                    System.out.println("There is"
                            +
                            " an untracked file in the way;"
                                    +
                                    " delete it, or add"
                                    +
                                    " and commit it first.");
                    return;
                }
            }
        }



        setActive(branchName);
        clearStagingArea();
        clearRemoveArea();

        for (int i = 0; i < prevFiles.length; i++) {
            File chopped = new File(CWD, prevFiles[i].getName());
            if (chopped.exists()) {
                chopped.delete();
            }
        }


        for (int i = 0; i < newFiles.length; i++) {
            File adding = new File(CWD, newFiles[i].getName());
            if (!adding.getName().equals("thisCommitObject")) {
                copyTo(getActiveCommit
                        (getActiveBranch()), CWD, adding.getName());
            }
        }

    }

    public static Commit getParentCommit(Commit c) {
        File childFile = c.getParent();
        return getCommitFromChildFile(childFile);
    }

    public static Commit getCommitFromChildFile(File f) {
        Commit c = Commit.readCommit(f.getName());
        return c;
    }

    public static void logPrint(Commit c) {
        System.out.println("===");
        System.out.println("commit " + c.getHash());
        System.out.println("Date: " + c.getTimeFormatted());
        System.out.println(c.getMessage());
        System.out.println();
    }

    public static void log() {
        boolean hasParent = true;

        Commit c = Commit.readCommit
                (getActiveCommit(getActiveBranch()).getName());
        while (hasParent) {
            hasParent = false;
            logPrint(c);
            if (c.getParent() != null) {
                c = getParentCommit(c);
                hasParent = true;
            }

        }

    }

    public static void globalLog() {
        File[] allCommitFolders = listOfFiles(COMMIT_FOLDER);
        for (int i = 0; i < allCommitFolders.length; i++) {
            Commit c = Commit.readCommit(allCommitFolders[i].getName());
            logPrint(c);
        }

    }

    public static void find(String commitMessage) {
        File[] allCommitFolders = listOfFiles(COMMIT_FOLDER);
        boolean foundAny = false;
        for (int i = 0; i < allCommitFolders.length; i++) {
            Commit c = Commit.readCommit(allCommitFolders[i].getName());
            if (c.getMessage().equals(commitMessage)) {
                System.out.println(c.getHash());
                foundAny = true;
            }
        }
        if (!foundAny) {
            System.out.println("Found no commit with that message.");
        }
    }

    public static boolean compare(File first, File second) {

        Diff d = new Diff();
        d.setSequences(first, second);
        return d.sequencesEqual();
    }

    public static void printAllExceptString(String s, File f) {
        File[] branches = listOfFiles(f);
        ArrayList<String> branchNames = new ArrayList<>();
        for (int i = 0; i < branches.length; i++) {
            if (!branches[i].getName().equals(s)) {
                branchNames.add(branches[i].getName());
            }
        }
        Collections.sort(branchNames);
        for (int i = 0; i < branchNames.size(); i++) {
            System.out.println(branchNames.get(i));
        }
    }

    public static void
        printAllExceptStringBranches(String s, File f) {
        File[] branches = listOfFiles(f);
        ArrayList<String> branchNames = new ArrayList<>();
        for (int i = 0; i < branches.length; i++) {
            if (!branches[i].getName().equals(s)) {
                branchNames.add(branches[i].getName());
            }
        }
        Collections.sort(branchNames);
        for (int i = 0; i < branchNames.size(); i++) {
            if (branchNames.get(i).equals
                    (Utils.readContentsAsString(ACTIVE))) {
                System.out.print("*");
            }

            System.out.println(branchNames.get(i));
        }
    }

    public static void status() {
        System.out.println("=== Branches ===");
        printAllExceptStringBranches("Active", BRANCH_FOLDER);
        System.out.println();
        System.out.println("=== Staged Files ===");
        printAllExceptString("", STAGING_FOLDER);
        System.out.println();
        System.out.println("=== Removed Files ===");
        printAllExceptString("", REMOVE_FOLDER);
        System.out.println();

        System.out.println("=== Modifications Not Staged For Commit ===");
        System.out.println();
        System.out.println("=== Untracked Files ===");
        System.out.println();

    }

    public static void branch(String branchName) throws IOException {
        File[] existingBranches = listOfFiles(BRANCH_FOLDER);
        for (File f : existingBranches) {
            if (f.getName().equals(branchName)) {
                System.out.println("A branch with that name already exists.");
                return;
            }
        }
        Branch b = new Branch(getActiveCommit
                (getActiveBranch()), branchName);
        b.writeBranch();

    }

    public static void removeBranch(String branchName) {
        if (getActiveBranch().getName().equals(branchName)) {
            System.out.println("Cannot remove the current branch.");
            return;
        }
        File branchCut = new File(BRANCH_FOLDER, branchName);
        if (!branchCut.exists()) {
            System.out.println("A branch with that name does not exist.");
            return;
        }
        branchCut.delete();

    }

    public static File latestCommonAncestor(Commit c) {
        TreeMap<String, Integer> referenceMap
                = Commit.readCommit(getActiveCommit
                (getActiveBranch()).getName()).getMap();
        TreeMap<String, Integer> branchMap = c.getMap();
        int min = Integer.MAX_VALUE;
        String chosen = null;
        Set<String> possibleIDs = branchMap.keySet();
        for (String iD : possibleIDs) {
            if (referenceMap.containsKey(iD)) {
                int distance = referenceMap.get(iD);
                if (distance < min) {
                    min = distance;
                    chosen = iD;
                }
            }
        }
        File chosenFile = new File(COMMIT_FOLDER, chosen);
        if (!chosenFile.exists()) {
            throw new RuntimeException("S");
        }
        return chosenFile;
    }

    public static void thisIsAtestingMethod(Branch b) {
        Commit test = Commit.readCommit
                (getActiveCommit(getActiveBranch()).getName());
        TreeMap<String, Integer> testMap = test.getMap();
        for (Map.Entry<String, Integer> entry : testMap.entrySet()) {
            System.out.println(entry.getKey()
                    + " has a value of " + entry.getValue());
            Commit speak = Commit.readCommit(entry.getKey());
            System.out.println(speak.getMessage());
        }


        Commit c = Commit.readCommit(latestCommonAncestor
                (Commit.readCommit(b.getPointer().getName())).getName());
        System.out.println(c.getMessage());
    }

    public static boolean mergeChecks(Branch chosenBranch) {
        Branch activeBranch = getActiveBranch();
        File activeFile = getActiveCommit(getActiveBranch());
        File chosenFile = chosenBranch.getPointer();
        Commit chosenCommit = Commit.readCommit(chosenFile.getName());
        File splitPointFile = latestCommonAncestor(chosenCommit);
        Commit test = Commit.readCommit(splitPointFile.getName());
        ArrayList<String> thingsNeedinAMerge = new ArrayList<>();

        boolean conflictEncountered = false;

        if (activeFile.getAbsolutePath().equals
                (chosenFile.getAbsolutePath())) {
            System.out.println("Cannot merge a branch with itself.");
            return true;
        }

        if (splitPointFile.getAbsolutePath()
                .equals(chosenFile.getAbsolutePath())) {
            System.out.println("Given branch is"
                    +
                    " an ancestor of the current branch.");
            return true;
        }

        if (splitPointFile.getAbsolutePath()
                .equals(activeFile.getAbsolutePath())) {
            System.out.println("Current branch fast-forwarded.");
            return true;
        }
        return false;
    }


    public static void merge(Branch chosenBranch)
            throws IOException, NoSuchAlgorithmException {
        Branch activeBranch = getActiveBranch();
        File activeFile = getActiveCommit(getActiveBranch());
        File chosenFile = chosenBranch.getPointer();
        Commit chosenCommit = Commit.readCommit(chosenFile.getName());
        File splitPointFile = latestCommonAncestor(chosenCommit);
        Commit test = Commit.readCommit(splitPointFile.getName());
        ArrayList<String> thingsNeedinAMerge = new ArrayList<>();
        boolean conflictEncountered = false;
        if (mergeChecks(chosenBranch)) {
            return;
        }
        File[] filesInSplit = listOfFiles(splitPointFile);
        File[] filesInActive = listOfFiles(activeFile);
        File[] filesInBranch = listOfFiles(chosenFile);
        for (int i = 0; i < filesInBranch.length; i++) {
            File workinInBranch = filesInBranch[i];
            File workinInActive =
                    new File(activeFile, workinInBranch.getName());
            File workinInSplit =
                    new File(splitPointFile, workinInBranch.getName());
            if (allInstancesDifferent
                    (chosenFile, activeFile,
                            splitPointFile, workinInBranch.getName())) {
                thingsNeedinAMerge.add(workinInBranch.getName());
            }
            if (!workinInActive.exists() && !workinInSplit.exists()) {
                checkout(workinInBranch.getName(), chosenFile);
            } else if (workinInSplit.exists()
                    && workinInActive.exists()
                    && workinInBranch.exists()) {
                boolean changedInBranch = false;
                boolean changedInActive = false;
                if (!compare(workinInSplit, workinInBranch)) {
                    changedInBranch = true;
                }
                if (!compare(workinInSplit, workinInActive)) {
                    changedInActive = true;
                }
                if (!changedInActive && changedInBranch) {
                    checkout(workinInBranch.getName(), chosenFile);
                }
            }
        }
        ArrayList<String> thing = isItOver(filesInSplit,
                activeFile, chosenFile, splitPointFile);
        for (String s : thing) {
            thingsNeedinAMerge.add(s);
        }
        thingsNeedinAMerge = removeDuplicates(thingsNeedinAMerge);
        for (String needs : thingsNeedinAMerge) {
            conflictEncountered = true;
            conflictMerge(chosenFile, activeFile, needs);
        }
        if (conflictEncountered) {
            System.out.println("Encountered a merge conflict.");
        }
        commitForMerge(activeBranch, chosenBranch);
        clearAreas();
    }

    public static void clearAreas() {
        clearStagingArea();
        clearRemoveArea();
    }

    public static ArrayList<String>
        isItOver(File[] filesInSplit, File activeFile, File chosenFile,
             File splitPointFile) throws IOException {
        ArrayList<String> toReturn = new ArrayList<>();
        for (int i = 0; i < filesInSplit.length; i++) {
            File workinInSplit = filesInSplit[i];
            File workinInActive = new File(activeFile, workinInSplit.getName());
            File workinInBranch = new File(chosenFile, workinInSplit.getName());

            if (allInstancesDifferent
                    (chosenFile, activeFile,
                            splitPointFile, workinInSplit.getName())) {
                toReturn.add(workinInSplit.getName());
            }
            if (workinInActive.exists() && !workinInBranch.exists()) {

                if (compare(workinInSplit, workinInActive)) {
                    remove(workinInSplit.getName());
                }
            }
        }
        return toReturn;
    }

    public static int doNothing() {
        int i = 0;
        return i;
    }

    public static void conflictMerge
    (File branch, File active, String target) throws IOException {
        File workinInActive = new File(active, target);
        File workinInBranch = new File(branch, target);
        String activeText;
        String branchText;
        if (workinInActive.exists()) {
            activeText = Utils.readContentsAsString(workinInActive);
        } else {
            activeText = "";
        }
        if (workinInBranch.exists()) {
            branchText = Utils.readContentsAsString(workinInBranch);
        } else {
            branchText = "";
        }


        String newConflictedFileContents = "<<<<<<< HEAD\n" + activeText
                + "=======\n" + branchText + ">>>>>>>\n";


        File putThisConflictedStuff = new File(STAGING_FOLDER, target);
        Utils.writeContents(putThisConflictedStuff, newConflictedFileContents);
        File conflictedStuffToCWD = new File(CWD, target);
        if (!conflictedStuffToCWD.exists()) {
            conflictedStuffToCWD.createNewFile();
        }
        Utils.writeContents(conflictedStuffToCWD, newConflictedFileContents);
    }

    public static boolean allInstancesDifferent
    (File branch, File active, File split, String target) {
        File first = new File(branch, target);
        File second = new File(active, target);
        File third = new File(split, target);

        if (!compare(first, second) && !compare(second,
                third) && !compare(first, third)) {
            return true;
        }
        return false;
    }

    public static File commitForMerge(Branch active, Branch chosen)
            throws IOException, NoSuchAlgorithmException {
        String commitMessage = "Merged "
                + chosen.getName() + " into " + active.getName()
                + ".";
        File f;
        Date d = new Date();
        Commit c = null;
        if (commitMessage.equals("initial commit")) {
            c = new Commit();
            f = makeFolderWithMessage(c.getHash());
            c.writeCommit();
        } else {

            c = new Commit(getActiveCommit(active),
                    getActiveCommit(chosen), d, commitMessage);
            f = makeFolderWithMessage(c.getHash());
            Branch b = getActiveBranch();
            File activeCom = getActiveCommit(b);

            File[] removeFiles = listOfFiles(REMOVE_FOLDER);
            for (int i = 0; i < removeFiles.length; i++) {
                File chopped =
                        getFileFromFolder(activeCom, removeFiles[i].getName());
                if (chopped.exists()) {
                    chopped.delete();
                }
            }

            File[] old = activeCom.listFiles();

            for (int i = 0; i < old.length; i++) {
                File curr = old[i];
                if (!curr.getName().equals("thisCommitObject")) {
                    File inRemoveFolder =
                            new File(REMOVE_FOLDER, curr.getName());
                    if (!inRemoveFolder.exists()) {
                        copyTo(activeCom, f, curr.getName());
                    }
                }
            }

            b.setPointer(f);
            b.writeBranch();
            c.writeCommit();

        }


        File[] files = listOfFiles(STAGING_FOLDER);

        for (int i = 0; i < files.length; i++) {
            File curr = files[i];

            copyTo(STAGING_FOLDER, f, curr.getName());
        }
        return f;
    }

    public static ArrayList<String> removeDuplicates(ArrayList<String> a) {
        ArrayList<String> answer = new ArrayList<>();
        for (String elem : a) {
            if (!answer.contains(elem)) {
                answer.add(elem);
            }
        }
        return answer;
    }

    public static void init()
            throws IOException, NoSuchAlgorithmException {
        if (REPO_FOLDER.exists()) {
            System.out.println("Gitlet versio"
                    +
                    "n-control system already"
                    +
                    " exists in the current directory.");
            return;
        }
        setupPersistence();
        initialize();

    }

    public static void reseto(String[] args)
            throws IOException {
        File[] allComms = listOfFiles(COMMIT_FOLDER);
        boolean isReal = false;
        for (File f : allComms) {
            if (f.getName().equals(args[1])) {
                isReal = true;
            }
        }
        if (!isReal) {
            System.out.println("No commit with that id exists.");
            return;
        }
        reset(args[1]);
    }
    public static void findo(String[] args) {
        String findStr = new String("");
        for (int i = 1; i < args.length; i++) {
            findStr = findStr + args[i] + " ";
        }
        findStr = findStr.substring(0, findStr.length() - 1);
        find(findStr);
    }

    public static void checkouto(String[] args)
            throws IOException {
        if (args.length == 3) {
            checkout(args[2], getActiveCommit(getActiveBranch()));
        } else if (args.length == 4) {
            if (!args[2].equals("--")) {
                System.out.println("Incorrect operands.");
                return;
            }

            String commitId = args[1];


            if (commitId.length() < 15) {
                File[] allCommitId = listOfFiles(COMMIT_FOLDER);
                for (File com : allCommitId) {
                    String longName = com.getName();
                    String shortName
                            = longName.substring(0, commitId.length());

                    if (shortName.equals(commitId)) {
                        commitId = longName;

                    }
                }
            }

            checkout(args[3], commitId);

        } else if (args.length == 2) {


            File newBranchFile = new File(BRANCH_FOLDER, args[1]);
            if (getActiveBranch().getName().equals(args[1])) {
                System.out.println("No ne"
                        +
                        "ed to checkout the current branch.");
                return;
            }

            if (!newBranchFile.exists()) {
                System.out.println("No such branch exists.");
                return;
            }
            checkoutBranch(args[1]);

        } else {
            throw new GitletException("wrong number of args");
        }
    }
    public static void commito(String[] args)
            throws IOException, NoSuchAlgorithmException {
        if (listOfFiles(STAGING_FOLDER).length
                + listOfFiles(REMOVE_FOLDER).length == 0) {
            System.out.println("No changes added to the commit.");
            return;
        }

        if (args[1].equals("")) {
            System.out.println("Please enter a commit message.");
            return;
        }

        String str = new String("");
        for (int i = 1; i < args.length; i++) {
            str = str + args[i] + " ";
        }

        str = str.substring(0, str.length() - 1);

        commit(str);
        clearStagingArea();
        clearRemoveArea();


    }

    public static boolean checkinStuff1(String[] args) {
        Branch br = Branch.readBranch(args[1]);
        File newHead = getActiveCommit(getActiveBranch());
        File oldHead = getActiveCommit(br);

        File[] makeSureTracked = newHead.listFiles();
        for (File newFile : makeSureTracked) {
            File problemIfNotExist = new File(oldHead, newFile.getName());
            File cwdFile = new File(CWD, newFile.getName());
            if (cwdFile.exists()) {
                if (!compare(cwdFile, newFile)) {
                    if (!problemIfNotExist.exists()) {
                        System.out.println("There is "
                                +
                                "an untracked file in the w"
                                +
                                "ay; delete it, or"
                                +
                                " add and commit it first.");
                        return true;
                    } else if (!compare(problemIfNotExist, cwdFile)) {
                        System.out.println("There is an untracked file"
                                +
                                " in the way; delete it, "
                                +
                                "or add and commit it first.");
                        return true;
                    }
                }
            }
        }
        return false;
    }

    public static boolean checkinStuff2(String[] args) {
        Branch br = Branch.readBranch(args[1]);
        File newHead = getActiveCommit(getActiveBranch());
        File oldHead = getActiveCommit(br);

        File[] makeSureTracked2 = oldHead.listFiles();
        for (File newFile : makeSureTracked2) {
            File problemIfNotExist = new File(newHead, newFile.getName());
            File cwdFile = new File(CWD, newFile.getName());
            if (cwdFile.exists()) {
                if (!compare(cwdFile, newFile)) {
                    if (!problemIfNotExist.exists()) {
                        System.out.println("There is an untracked fil"
                                +
                                "e in the way; delete it"
                                +
                                ", or add and commit it first.");
                        return true;
                    } else if (!compare(problemIfNotExist, cwdFile)) {
                        System.out.println("There is an untracked file"
                                +
                                " in the way; delete it,"
                                +
                                " or add and commit it first.");
                        return true;
                    }
                }
            }
        }
        return false;
    }
    public static boolean fakeMain(String[] args)
            throws IOException, NoSuchAlgorithmException {
        switch (args[0]) {
        case "init":
            init();
            return true;
        case "add":
            add(args);
            return true;
        case "commit":
            commito(args);
            return true;
        case "checkout":
            checkouto(args);
            return true;
        case "log":
            log();
            return true;
        case "global-log":
            globalLog();
            return true;
        case "rm":
            remove(args[1]);
            return true;
        case "status":
            status();
            return true;
        case "find":
            findo(args);
            return true;
        case "branch":
            branch(args[1]);
            return true;
        case "rm-branch":
            removeBranch(args[1]);
            return true;
        default:
            return false;
        }

    }



}

+++
title = 'Prompt Engineering'
date = 2025-08-09T11:00:25-04:00
draft = true
+++

One weird thing about AI alignment is that the AI will often match your level
of thoroughness and understanding. Here's an example of what I mean:

```
Okay, that worked. Great job!

[x] Split up cell_allocator.h implementation into .c file

Now,

[] Do not pre-allocate cells

For this task, currently in my allocator, I am allocating every cell of the spreadsheet _in advance_. This is fine if it's just a 10x10 grid, but will become wasteful fast when I eventually want the spreadsheet to feel infinite / allocate as needed. For instance, I should be able to edit a single cell at ZZZZ100000 without a problem. The philosophy behind this is that we want the problem to roughly be the same shape as a human mind. It's not enough to say, hey, no human will realistically ever need a million cells. Instead, the computer should optimize around the hard limit of brain ram which is a _complexity limit_, not a hard cell count limit. In other words, the human should gas out before the computer. And of course, if you're running automatic calculations on large datasets, that's a different story, but I don't want anyone to see the cracks when someone like me decides to push it. Does that make sense?

Anyway, in order to not pre-allocate cells, I need a way to collect cells with empty formulas and allocate new ones only if they don't already exist in my BST of BSTs. This is kind of unpleasant because I have to do a log(n) search, see that it doesn't exist, and then create it. When in reality I should be able to write a function which traverses the tree, creates and inserts it if it doesn't exist, and then return it, like a default constructible map. Does that make sense? My current design is a little janky in that way. As a reminder, here's some of the code:

'''
// Cell Tree Structures

typedef struct ColNode
{
    Cell *cell;
    struct ColNode *left, *right;
} ColNode;

typedef struct RowNode
{
    int row;
    ColNode *col_tree;
    struct RowNode *left, *right;
} RowNode;

static RowNode *row_root = NULL;

// --- Utility functions ---

static Cell *make_cell(int row, int col, int value)
{
    Cell *cell = (Cell *)alloc_cell();
    if (!cell)
        return NULL;
    cell->row = row;
    cell->col = col;
    cell->value = value;
    return cell;
}

// --- BST helpers ---

static ColNode *col_insert(ColNode *node, Cell *cell)
{
    if (!node)
    {
        ColNode *n = (ColNode *)alloc_cell();
        if (!n)
            return NULL;
        n->cell = cell;
        n->left = n->right = NULL;
        return n;
    }
    if (cell->col < node->cell->col)
        node->left = col_insert(node->left, cell);
    else if (cell->col > node->cell->col)
        node->right = col_insert(node->right, cell);
    return node;
}

static ColNode *col_delete(ColNode *node, int col, Cell **freed)
{
    if (!node)
        return NULL;
    if (col < node->cell->col)
        node->left = col_delete(node->left, col, freed);
    else if (col > node->cell->col)
        node->right = col_delete(node->right, col, freed);
    else
    {
        if (freed)
            *freed = node->cell;
        if (!node->left)
        {
            ColNode *r = node->right;
            dealloc_cell(node);
            return r;
        }
        if (!node->right)
        {
            ColNode *l = node->left;
            dealloc_cell(node);
            return l;
        }
        ColNode *succ = node->right;
        while (succ->left)
            succ = succ->left;
        node->cell = succ->cell;
        node->right = col_delete(node->right, succ->cell->col, NULL);
    }
    return node;
}

static void col_range_query(ColNode *node, int c1, int c2, void (*visit)(Cell *))
{
    if (!node)
        return;
    if (c1 <= node->cell->col)
        col_range_query(node->left, c1, c2, visit);
    if (node->cell->col >= c1 && node->cell->col <= c2)
        visit(node->cell);
    if (node->cell->col <= c2)
        col_range_query(node->right, c1, c2, visit);
}

static RowNode *row_insert(RowNode *node, Cell *cell)
{
    if (!node)
    {
        RowNode *n = (RowNode *)alloc_cell();
        if (!n)
            return NULL;
        n->row = cell->row;
        n->col_tree = col_insert(NULL, cell);
        n->left = n->right = NULL;
        return n;
    }
    if (cell->row < node->row)
        node->left = row_insert(node->left, cell);
    else if (cell->row > node->row)
        node->right = row_insert(node->right, cell);
    else
        node->col_tree = col_insert(node->col_tree, cell);
    return node;
}

static RowNode *row_delete(RowNode *node, int row, int col)
{
    if (!node)
        return NULL;
    if (row < node->row)
        node->left = row_delete(node->left, row, col);
    else if (row > node->row)
        node->right = row_delete(node->right, row, col);
    else
    {
        Cell *freed = NULL;
        node->col_tree = col_delete(node->col_tree, col, &freed);
        if (!node->col_tree)
        {
            if (!node->left)
            {
                RowNode *r = node->right;
                dealloc_cell(node);
                return r;
            }
            if (!node->right)
            {
                RowNode *l = node->left;
                dealloc_cell(node);
                return l;
            }
            RowNode *succ = node->right;
            while (succ->left)
                succ = succ->left;
            node->row = succ->row;
            node->col_tree = succ->col_tree;
            node->right = row_delete(node->right, succ->row, -1);
        }
    }
    return node;
}

static void row_range_query(RowNode *node, int r1, int r2, int c1, int c2, void (*visit)(Cell *))
{
    if (!node)
        return;
    if (r1 <= node->row)
        row_range_query(node->left, r1, r2, c1, c2, visit);
    if (node->row >= r1 && node->row <= r2)
        col_range_query(node->col_tree, c1, c2, visit);
    if (node->row <= r2)
        row_range_query(node->right, r1, r2, c1, c2, visit);
}

// --- public API ---

Cell *create_cell(int row, int col, int value)
{
    Cell *c = make_cell(row, col, value); // @ChatGPT problem is here -- node is constructed
    if (!c)
        return NULL;
    row_root = row_insert(row_root, c); // pre-created node is inserted -- we want something like find_or_create_cell()
    return c;
}

void delete_cell(int row, int col)
{
    row_root = row_delete(row_root, row, col);
}

void query_cells(int r1, int c1, int r2, int c2, void (*visit)(Cell *))
{
    row_range_query(row_root, r1, r2, c1, c2, visit);
}
'''

The new function that we want doesn't even have to take a value -- that can be modified after the fact, once we have the pointer. The main thing is we want that default-constructible map energy, you feel?
```

This prompt was special because it led to a complete, concise, and bug-free implementation of the function
`get_or_create_cell`, which I was not looking forward to having to write.

Here's what I identify makes this a good prompt:
1. There's a clear context and progression: it starts by acknowledging completed work, ("[x] Split up 
    cell_allocator.h
    implementation") and then clearly states the next objective. This gives the AI the full picture
    of where things stand.
2. Well-articulated problem statement: although I'd say I was waxing a little too poetic about spreadsheet UX,
    I think that explaining the underlying philosophy framed the problem in a nice way, and perhaps somehow
    activated whichever neurons are responsible for elegance within the LLM.
3. Technical precision: there's domain specific terimnology ("BST of BSTs", "log(n) search", "default constructible map")
    which signals the level of understanding that I have and elicits a more sophisticated response.
4. Code context: This one is interesting. LLMs are scarily good at refactoring in one shot, provided you've given them
    an already working implementation. It provides a scaffolding for the LLM to work off of. Providing actual code,
    instead of pseudocode, or just asking the LLM to one-shot greatly improves the LLM's performance.
5. Annotating the code example: there's a clear annotation pointing to the specific problem area ("@ChatGPT problem is here
    -- node is constructed").
6. Specifying desired outcome: rather than leaving the solution open-ended, I specify "something like `find_or_create_cell()`",
    a clear, implementable pattern with a descriptive name.
7. Conversational calibration: Phrases like "does that make sense?", and "you feel", I think frame the conversation 
    as a problem solving session. This allows the LLM to behave more fluidly, by exercising its own judgement, providing
    only code it is confident about and asking for clarification instead of forcing the LLM to do its best with a 
    suboptimal understanding. Also, I think this boosts LLM morale, which, from what I've read, is a real thing.

Claude puts it best: "when someone puts in the effort to fully contextualize a problem, use precise language, and show 
their current thinking, they get responses that match that level of care and technical depth."
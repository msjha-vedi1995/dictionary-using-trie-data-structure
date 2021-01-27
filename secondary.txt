#include <iostream>
#include <bits/stdc++.h>
#include<string>
#include<stdio.h>
#include<stdlib.h>
using namespace std;
struct Trie {                            //structure of trie
    bool isEndOfWord;                    //boolean used to determine whether respective node is end of node or not
    unordered_map<char, Trie*> map_1;    //unordered map used to connect one trie to another(character of word connected to next connector)
    string meaning;                      //Meaning of word stored at last node(i.e word is complete)
    vector<char> letters;                //vector(dynamic character array) used to store characters in node(used for auto-suggest feature)
};


Trie* getNewTrieNode()                   //function to create new trie node
{
    Trie* node = new Trie;
    node->isEndOfWord = false;
    return node;
}

void insert(Trie*& root,string str,string meaning){ //function to insert nodes

    if (root == NULL){
        root = getNewTrieNode();
    }

    Trie* temp = root;
    for (int i = 0; i < str.length(); i++) {
        char x = str[i]; //x used to insert character into trie data structure
        if(x>=65&&x<=90)//converting character to lower case if upper case characters present. Easy to work with all letters of one case, in this case lower case
            x=x+32;
        if (temp->map_1.find(x) == temp->map_1.end()){//if letter is not present while traversing trie, new trie node is created and connected to previous trie node using unordered map
            temp->map_1[x] = getNewTrieNode();
            temp->letters.push_back(x);
        }
        temp = temp->map_1[x];  //temp is incremented to next trie node using unordered map
    }
    temp->isEndOfWord = true; //setting boolean isEndOfWord true because last trie node(leaf node) will always be last character of word and determines whether word is complete or not
    temp->meaning = meaning;  //meaning of word added in last trie node
}

void autosuggest(Trie* root,const string query,int flag){//Simple auto-suggest function
    string suggested_word="";
    Trie* temp=root;
    int i;
    for(i=0;i<query.length();i++){//traversing query user entered
        suggested_word.push_back(query[i]);
        temp=temp->map_1[query[i]];
    }
    char x;
    while(!temp->isEndOfWord){//suggesting words that include user's query
        int recursive_flag=++flag;
        x=temp->letters.at(0);
        if(temp->letters.size()>=recursive_flag){
            autosuggest(root,query,recursive_flag);
        }
        if((temp->letters.size()>=flag)&&(flag>1)){
            x=temp->letters.at(flag-1);
        }
        suggested_word.push_back(x);
        temp=temp->map_1[x];
    }
    cout<<suggested_word<<endl;
}

string getMeaning(Trie* root,string word)  //function used to get meaning of word user entered
{
    if (root == NULL)  //if root is empty,trie is completely empty
        return "word not present";

    Trie* temp = root;

    for (int i = 0; i < word.length(); i++) { //traversing the trie
        temp = temp->map_1[word[i]];
        if (temp == NULL)
            return"word not present";
    }

    if (temp->isEndOfWord){                 //boolean isEndOfWord is true i.e last node has been reached and word exists
        cout<<"Meaning is: "<<endl;
        return temp->meaning;
    }
    cout<<"Word not present. Did you mean: "<<endl;
    autosuggest(root,word,0);                 //if word not present, auto-suggest function is called to find nearest word present in the dictionary that includes user's query as prefix
    return"";
}
void search(Trie* root){ //function to take input from user and search for word
    cout<<"Enter a word"<<endl; //taking input from user
    string word;
    cin>>word;
    int s;
    cout<<getMeaning(root,word)<<endl;
    cout<<"\nEnter 0 to search again. "<<endl;
    cin>>s;
    if(s==0){
        search(root);
    }
    else if(s!=0){
        exit(0);
    }
}


int main(){
    Trie* root=NULL;

    ifstream myFile;
    string line;

    myFile.open("dictionary.txt"); //opening file with all words of english language

    while(myFile.good()){//using file handling to insert all words into trie structure
        getline(myFile,line);
        int a=line.find("\"");
    int b=line.find("\"",a+1);
    string word="";
    string meaning="";

    for(int i=a+1;i<b;i++){//extracting word from line
        word.push_back(line[i]);
    }

    int a_1=line.find("\"",b+1);
    int b_1=line.find("\"",a_1+1);
    for(int j=a_1+1;j<b_1;j++){//extracting meaning from line
        meaning.push_back(line[j]);
    }
    insert(root,word,meaning);
    }
    search(root);

}

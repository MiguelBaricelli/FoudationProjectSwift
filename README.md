# FoudationProjectSwift



## Project

//
//  ContentView.swift
//  Project-pokemon-manha
//
//  Created by Mack Aluno on 29/07/25.
//
import Foundation
import SwiftUI

struct ContentView: View {
    
   @State private var url = "https://pokeapi.co/api/v2/"
    
    var body: some View {
    
 
        TabView {
            
            ZStack {
                
                Spacer(minLength: 40)
                
                VStack {
                    Text("POKEMONS")
                        .foregroundColor(.gray)
                        .fontWeight(.bold)
                    
                        
                    VStack {
                        
                    
                        HStack {
                            
                            RoundedRectangle(cornerRadius: 10)
                            Image(systemName: "gear")
                                .frame(width: 80, height: 80)
                                .background(.gray)
                            
                            RoundedRectangle(cornerRadius: 10)
                            Image(systemName: "gear")
                                .frame(width: 80, height: 80)
                                .background(.gray)
                            
                            
                            RoundedRectangle(cornerRadius: 10)
                            Image(systemName: "gear")
                                .frame(width: 80, height: 80)
                                .background(.gray)
                            
                            
                            
                            RoundedRectangle(cornerRadius: 10)
                            Image(systemName: "gear")
                                .frame(width: 80, height: 80)
                                .background(.gray)
                            
                            
                            
                        }
                        
                        
                        
                    }
                    
                }
                
                
            
                
            }
            
            
            
        
            
            
            .tabItem { Label("Menu", systemImage: "house") }.tag(1)
            Text("Tab Content 2")
                .tabItem { Label("Estatisticas", systemImage: "chart.line.uptrend.xyaxis") }.tag(2)
            Text("Tab Content 3")
                .tabItem { Label("Pesquisar", systemImage: "magnifyingglass.circle.fill") }.tag(3)
        }
        .padding()
    }
        
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}


public class Pokemon() {
    /*
     tratar informacoes api
     trazer nome, url, habilidade ...
     
     funcao(id) -> [nome : imagem] { botao da home }
     
     funcao pegar(id)  { muda o status }
     */
    
    var
}

